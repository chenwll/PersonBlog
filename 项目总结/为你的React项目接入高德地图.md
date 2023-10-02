最近在写一个项目的时候，需要使用地图来选取点位信息

# 使用高德地图
在技术选型上面，我们使用Antd推荐的高德地图，并支持React的组件化
```js
npm install --save react-amap
```
去[高德地图](https://console.amap.com/dev/key/app)申请key和密钥


![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a1ebecd473cc4e27a5f14e8a7346d9c4~tplv-k3u1fbpfcp-watermark.image?)

## 添加key和安全密钥

### 给Map添加key
```js
<Map
  amapkey={MAP_KEY}
  plugins={plugins}
  events={this.handleMapEvents}
  zoom={17}
  center={[lng, lat]}
  doubleClickZoom={false}
>
```
### 添加安全密钥
我们在开发环境下直接使用`script`标签添加安全密钥
```js
<script type="text/javascript"> 
    window._AMapSecurityConfig = { securityJsCode:'您申请的安全密钥', } 
</script>
```
在React脚手架中，我们可以在`index.html`文件中添加script标签。但是我们使用的是Umi项目，没有`index.html`，我们可以更改Umi默认模板，详见：[修改默认模板](https://v2.umijs.org/zh/guide/html-template.html#%E4%BF%AE%E6%94%B9%E9%BB%98%E8%AE%A4%E6%A8%A1%E6%9D%BF)

1. 在pages文件夹下面添加`document.ejs`文件
2. 添加密钥

# 创建地图组件
我们最后实现的效果可能会是这个样子

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f83bcad3d9c84bd48ef459392f2ebd5c~tplv-k3u1fbpfcp-watermark.image?)

我们要实现的部分主要分成三个

1. 搜索框的实现，包括搜索提示


![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/805cb670d9f64d42a1f76abd40afe7e7~tplv-k3u1fbpfcp-watermark.image?)

2. 地图展示部分，功能有根据根据经纬度进行定位，通过点击地图来逆编码来获得地址
3. 搜索结果的基本信息展示

我们在设计组件的时候，可以从功能上进行拆分，并考虑到组件复用的问题，尽量让组件原子化，只干一件事情。还可以将组件分成容器组件(只管理数据)和渲染组件(只管理视图)进行分离，并尽量保持组件的无状态，无state

## 划分组件

根据上面的分析，我们可以将搜索框单独设置为一个组件，并且可以复用在其他需要搜索地址的地方。将地图的展示单独设置成为一个组件，只需要接受到经纬度就好可以展示相应的信息了。搜索结果也可以单独展示出来就好了。

同时，在搜索框输入地址的时候，我们可以改变地图组件的状态；点击地图中的点位的时候，也可以改变输入框中的值。所以我们可以创建一个容器组件来管理两者互通的数据，即经纬度和地址名称

示意图：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eabef1294fc04130b280347a0054307a~tplv-k3u1fbpfcp-watermark.image?)

## 搜索组件编码
JavaScript API支持搜索服务脱离地图使用，即使用搜索服务不再需要先实例化地图。您可通`AMap.plugin`方法，加载需要的服务。同时JavaScript API将原有的通过事件监听获得服务查询结果，修改为通过方法的回调函数获得服务查询结果。

参考：https://lbs.amap.com/api/javascript-api/reference/search#m_AMap.PlaceSearch

由于我们每次改变输入框中的值，就会触发一次请求，所以我们可以在输入框中加一个防抖函数

```js
import React, { Component } from 'react';
import { Select } from 'antd';

const { Option } = Select;
class SearchAddress extends Component {
  constructor(props) {
    super(props);
    this.state = {
      positionList:[],
    }
    this.handleMapEvents =  {
      created: () => {
        window.AMap.plugin('AMap.PlaceSearch', () => {
          new window.AMap.PlaceSearch({
            pageSize: 10,
            pageIndex: 1,
          });
        })
      },
    }
  }



  onSearch =(val)=>{
    const place = new window.AMap.PlaceSearch({
      pageSize: 10,
      pageIndex: 1,
      city: '绵阳',
    })

    place.search(val,(status, result)=>{
      if(status === 'complete' && result.info === 'OK'){
        const {poiList:{pois}}=result
        if(pois && Array.isArray(pois)) {
          this.setState({
            positionList:pois
          })
        }
      }

    })
  }


  debounce = (fn,time) => {
    let timerId = null;
    return function (val) {
      if(timerId) {
        clearTimeout(timerId);
        timerId = null;
      }
      timerId = setTimeout(() => {
        fn.call(this,val)
      },time)
    }
  }
  
  // 选中Seleect框中触发回调，改变父组件的position和addressName
  onChange = (id) => {
    const { positionList } = this.state;
    const {changeAddressName，changePosition} = this.props;
    for(const item of positionList) {
      const { name:itemName,id:itemId } = item;
      if(itemId === id) {
        const {location : {lng,lat}} = item;
        const position = {lng,lat};
        changePosition(position)
        changeAddressName(itemName)
      }
    }
  }

  render() {
    const {addressName} = this.props
    const {positionList} = this.state;
    return (
      <div>
        <Select
          value={addressName}
          style={{ width: 400 }}
          showSearch
          placeholder="请输入地址"
          onSearch={this.debounce((val) => this.onSearch(val),300)}
          onChange={this.onChange}
          optionFilterProp="children"
        >
          {
            positionList.map( item=>
              <Option key={item.id} value={item.id}>{item.name}</Option>
            )
          }
        </Select>
      </div>
    );
  }
}

export default SearchAddress;
```

## 地图组件编码实现
当我们点击地图时，只能通过click事件得到经纬度，而得不到地址名。经纬度对于用户是没有用的，
所以我们使用逆编码来获得地址名称

参考： https://lbs.amap.com/api/javascript-api/reference/lnglat-to-address#m_AMap.Geocoder

我们还可以使用static propTypes和defaultProps来规定props传入值的类型和默认值
```js
static propTypes = {
  position:PropTypes.object,
  plugins: PropTypes.array,
};

static defaultProps = {
  position :{
    lng:104.679127,
    lat:31.467673,
  },
  plugins: ["ToolBar", 'Scale']
};
```
地图组件：
```js
import React, { Component, Fragment } from 'react';
import {Map, Marker} from 'react-amap';
import PropTypes from 'prop-types';
import { message } from 'antd';
import { MAP_KEY } from '@/utils/Enum';

class AdvancedMap extends Component {
  static propTypes = {
    position:PropTypes.object,
    plugins: PropTypes.array,
  };

  static defaultProps = {
    position :{
      lng:104.679127,
      lat:31.467673,
    },
    plugins: ["ToolBar", 'Scale']
  };

  constructor(props) {
    super(props);
    this.handleMapEvents =  {
      created: () => {
        window.AMap.plugin('AMap.PlaceSearch', () => {
          new window.AMap.PlaceSearch({
            pageSize: 10,
            pageIndex: 1,
          });
        })
        window.AMap.plugin('AMap.Geocoder', () =>{
          new window.AMap.Geocoder({
            city: '0816'
          })
        })
      },
      click: (e) => {
        const { lnglat:lnglatObj,lnglat: {lng,lat} } = e;
        const { changePosition,changeAddressName } = this.props;
        const lnglatArr = [lng,lat]
        const geocoder = new window.AMap.Geocoder({
          city: '0816'
        })
        changePosition(lnglatObj)
        geocoder.getAddress(lnglatArr, (status, result) => {
          if (status === 'complete' && result.info === 'OK') {
            console.log(result,'result');
            const { regeocode :{formattedAddress}} = result;
            changeAddressName(formattedAddress)
            message.success(formattedAddress)
          }
        })
      }
    }
  }


  render() {
    const {position: { lng,lat }, plugins} = this.props;
    return (
      <Fragment>
        <Map
          amapkey={MAP_KEY}
          plugins={plugins}
          events={this.handleMapEvents}
          zoom={17}
          center={[lng, lat]}
          doubleClickZoom={false}
        >
          <Marker position={[lng, lat]} />
        </Map>
      </Fragment>
    );
  }
}

export default AdvancedMap;
```
## 父级容器
使用父级容器主要是管理数据(position,addressName)，并传递改变数据(position,addressName)的方法给子组件，子组件调用父组件传递的函数，就可以更改父组件的数据(参考父子组件参数)

```js
import React, { Component } from 'react';
import { Col, Input, Row } from 'antd';

import SearchAddress from '@/pages/Map/SearchAddress';
import AdvancedMap from '@/pages/Map/AdvancedMap';

class Index extends Component {
  constructor(props) {
    super(props);
    this.state = {
      position: {
        lng:104.679127,
        lat:31.467673,
      },
      addressName:''
    }
  }

  changePosition = (value) => {
    this.setState({
      position:value
    })
  }

  changeAddressName = (value) => {
    this.setState({
      addressName:value
    })
  }
  
  render() {
    const {position,addressName} = this.state;
    return (
      <div style={{ width: '100%', height: '500px' }}>
        <Row gutter={24}>
          <Col span={8}>
            <SearchAddress
              changePosition={this.changePosition}
              changeAddressName={this.changeAddressName}
              addressName={addressName}
            />
          </Col>
        </Row>
        <br />
        <AdvancedMap
          position={position}
          changePosition={this.changePosition}
          changeAddressName={this.changeAddressName}
        />
      </div>
    );
  }
}

export default Index;
```

# 效果
后续还要再完善一下，比如规定好每个组件的输入输出，防止使用的时候出错

![文明城市.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2272a9a0d88240928c59d8720e6bfe3c~tplv-k3u1fbpfcp-watermark.image?)

# 第二版

实现海量点标记，并点击每一个点位的时候可以看到每个点位的具体信息
功能有：
1. 如果地图上已经该标记的点位了，展示相应信息
2. 如果没有该标记的点位，显示对应点位的信息并可以新增点位

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/47f59793f31a47a7a33fd40f73456a31~tplv-k3u1fbpfcp-watermark.image?)

## 海量点标记
海量点的数据肯定需要传到地图组件里面的，positions表示海量点的一个数据信息

我们拿到positions值的信息，创建Markers即可
```
<AdvancedMap
  positions={positions}
/>
```

## Marker信息展示
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1616018bf81143bd975463fd65a1f9d2~tplv-k3u1fbpfcp-watermark.image?)
这个页面的数据就比较复杂了，首先要展示的Form表单的结构肯定不能写死，这样复用性就很差，我们应该是通过容器组件来传入表单的结构，我们自动生成表单

表单的结构设计可以看之前的一篇：[关于我对表单设计的一点思考](https://juejin.cn/post/7203224726361063483)

```js
const formFields = [
  {
    "label": "首页图",
    "defaultValue": "",
    "field": "picture",
    "type": "string",
    "required": true,
    "pattern": null,
    "message":'Please upload picture',
    "disabled":true,
  },
  {
    "label": "经度",
    "defaultValue": "192.168.0.1",
    "field": "longitude",
    "type": "string",
    "required": true,
    "pattern":null,
    "message":'Please input',
    "disabled":true,
    "hidden":true
  },
  {
    "label": "纬度",
    "defaultValue": "21",
    "field": "latitude",
    "type": "string",
    "required": true,
    "pattern": null,
    "message":'Please input',
    "disabled":true,
    "hidden":true,
  },
  {
    "label": "活动名称",
    "defaultValue": "",
    "field": "activity",
    "type": "string",
    "required": true,
    "pattern":null,
    "message":'Please input',
    "disabled":false,
  },
  {
    "label": "活动地址",
    "defaultValue": "",
    "field": "address",
    "type": "string",
    "required": true,
    "pattern":null,
    "message":'Please input',
    "disabled":false,
  },
  {
    "label": "部门负责人",
    "defaultValue": "",
    "field": "responsibility",
    "type": "string",
    "required": true,
    "pattern":null,
    "message":'Please input',
    "disabled":false,
  },
  {
    "label": "联系方式",
    "defaultValue": "",
    "field": "phoneNumber",
    "type": "string",
    "required": true,
    "pattern":null,
    "message":'Please input',
    "disabled":false,
  },
  {
    "label": "部门介绍",
    "defaultValue": "",
    "field": "introduce",
    "type": "string",
    "required": true,
    "pattern":null,
    "message":'Please input',
    "disabled":false,
    "style":{
      "width":470,
      "height":110,
    }
  },
]

```
然后我们可以根据这些字段，动态生成一个表单
```js
buildFormItem = (formFields) => formFields.map((fields) => {
  const { form: { getFieldDecorator } } = this.props;
  const {field, label,required, type, pattern, message,disabled,style} = fields;
  const uploadButton = (
    <div>
      <Icon type="plus" />
      <div className="ant-upload-text">Upload</div>
    </div>
  );
  if(field === FORM_TYPE["picture"]) {
    return (
      <Form.Item label={label}>
        {getFieldDecorator(field, {
          rules: [{ required,message ,pattern,type}],
        })(
          <Upload
            className={style.upload}
            action="/campus/campusweb/upload/pictureUpload"
            accept={'image/*'}   // 接受图片的格式类型
            listType="picture-card"
            // fileList={} //用来存放上传图片的数组，放在状态里面
            name="multipartFile"
          >
            {uploadButton}
          </Upload>
        )}
      </Form.Item>
    )
  }
  return (
    <Form.Item label={label}>
      {getFieldDecorator(field, {
        rules: [{ required,message ,pattern,type}],
      })(
        <Input disabled={disabled} style={{ ...style }} />,
      )}
    </Form.Item>
  );
})
```

其次要思考的是，点击Marker后显示Form表单，这个事件应该是组件内部处理，还是容器组件处理呢？

我们在点击每个Marker的时候，都需要去显示点位信息，并且我们已经将每个点位信息作为一个数组传进去了，我们在点击Marker的时候会自动获取到我们点击的Marker的信息，没有必要再容器组件中控制表单的显示，也具有了更好的内聚性

```js
this.markersEvents = {
  created:() => {
    console.log('All Markers Instance Are Below');
  },
  click: (MapsOption, marker) => {
    // 获取信息
    const data = marker.getExtData();
    this.openModal(data)
  },
}
```
## 添加点位信息
我们是将点位信息做成一个positions数组传过去了，我们想要修改点位，只能通过修改positions数组来实现点位的新增

这时候我们就要在容器组件定义一个操作修改positions数组的方法，通过子组件调用父组件传递过来的函数，修改父组件中的值

那这个函数的调用时机是什么，当我们点击地图时，会展示相关信息，如果填写之后点击保存，我们才会新增点位。即`Modal`中的`onOK`事件

先判断有没有点位，再进行新增

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c7374a75734b4446a6f4188f2dd56e26~tplv-k3u1fbpfcp-watermark.image?)

# Api
## MarkerMap
| 参数       | 说明                                                         | 类型                        | 默认值 |
| ---------- | ------------------------------------------------------------ | --------------------------- | ------ |
| positions  | 当有多个点位信息需要展示时，使用positions代表所有点位的一个数组,包含每一个点位的信息 | object[]                    |        |
| marker     | 当只有一个点位信息时，使用marker                             | object                      |        |
| formFields | 点击marker时，所要展示的表单字段信息                         | object[]                    |        |
| clickMap   | 点击地图时触发该函数，可以获得点击位置的经纬度坐标和点击位置的地址 | Function(lnglatObj,address) |        |
| formVisible | 单击地图时是否显示相关表单信息 | Boolean | false
|getFormValue|获取表单的信息| Function(value) |

## SearchAddressInput
| 参数          | 说明                                     | 类型                        | 默认值 |
| ------------- | ---------------------------------------- | --------------------------- | ------ |
| addressName   | 搜索框中的默认值                         | String                      |        |
| chooseOneItem | 选择Select框中的一项数据时触发的回调函数 | Function(lnglatObj,address) |        |

源码还在整理中，就不公开了哈