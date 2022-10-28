---
title: vue项目路由模式为history使用微信jssdky分享总结
author: deephug
index_img: /img/wallhaven-e7zogk.jpg
category: 技术
tags:
  - Fluid
excerpt: 前端小菜鸟，因为项目要对接微信的jssdk，每回遇到微信分享都是一个坑，项目使用Vue开发，采用history的路由模式，对接就需要签名认证，但是无奈安卓和IOS各有各的坑，最后终于解决了，现将解决的过程分享一下。
date: 2022-10-26 16:06:45
---

## 调用微信 JSSDK 来实现分享功能

> 前端小菜鸟，因为项目要对接微信的jssdk，每回遇到微信分享都是一个坑，项目使用Vue开发，采用history的路由模式，对接就需要签名认证，但是无奈安卓和IOS各有各的坑，最后终于解决了，现将解决的过程分享一下。

## 闲话

公司有个需求要做微信分享，而且是全局的分享（每个页面都要有分享功能）功能。为了以后能方便点，就封装了一下。 **不是懒不是懒不是懒！！！重要的事情说三遍！！！**

## 技术要点

Vue, history路由

## 常见问题及说明

### debug模式下报false

这个没得说，就是调用wx.config方法的参数错误造成的，请确认以下事项：

是否成功绑定了域名（域名校验文件要能被访问到）
  - 1、使用最新的js-sdk文件，因为微信会改部分api
  - 2、config方法的参数是否传正确了（拼写错误、大小写...）
  - 3、需要使用的方法是否写在了**jsApiList**中
  - 4、获取签名的url需要**decodeURIComponent**
  - 5、后台的生成签名的加密方法需要对照官方文档

### debug返回ok，分享不成功

  - 1、确保代码拼写正确
  - 2、分享链接域名或路径必须与当前页面对应的公众号JS安全域名一致
  - 3、接口调用需要放在wx.ready方法中

### 路由传参的时候参数字符串里面不要带额外的 = 

这是什么意思呢？

在我的项目里面会有一些页面传参，有时候会传递 json 字符串，项目规定不可明文传输数据，所以又把 json字符串 ，通过base64.encode了一下，就导致了在url最后面会有连续两个 **==** ，然后这个页面的分享就失效了，直接 **debug的时候就false，报签名不通过** 。解决办法就是再将其 encodeURIComponent 一下，然后进行页面跳转

**巨坑巨坑巨坑，这个坑绝对是微信的巨坑，这个bug我测试了许久才测试出来**

### 签名

 - 安利一个在微信下能检测内核的判断：window.__wxjs_is_wkwebview(是否为webview内核)，如果是的情况下就变成true
 - 因为SPA应用下，会有一定的问题，路由采用的是history模式(不带#号)。因为在页面每次进入到路由之后才去获取签名授权，所以将方法公用写在路由的模块下3
 - window.entryUrl这个是什么鬼？这个是自己定义的全局属性，就是为了获取IOS第一次进入页面的时候存储起来的，如果IOS的签名的路径不是第一次进入的页面，那么就一定会失败，所以这个路由第一次进入就要储存起来
 - 为什么要写在router.afterEach，因为页面进入了再去做申请和签名，如果在beforeEach，会导致页面申请签名的时候还是上一个页面，但是到了新页面却没有注册签名，或者因为签名的路径不同，导致invalid signature
 - 为什么安卓的时候要增加一个延时器，因为安卓会存在一些情况，就是即便签名成功，但是还是会唤不起功能，这个貌似是一个比较稳妥的解决办法，
 

## 单页项目（SPA）中的一些要点

> 所有需要使用JS-SDK的页面必须先注入配置信息，否则将无法调用（同一个url仅需调用一次，对于变化url的SPA的web app可在每次url变化时进行调用,目前Android微信客户端不支持pushState的H5新特性，所以使用pushState来实现web app的页面会导致签名失败，此问题会在Android6.2中修复）。

上面那段话摘自[官方文档](https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/JS-SDK.html)

开发者需要注意的事项：

  - 1、android和ios需要分开处理
  - 2、需要在页面url变化的时候重新调用 **wx.config** 方法，android获取签名的url就传 **window.location.href**
  - 3、Vue项目在切换页面时，IOS中浏览器的url并不会改变，依旧是第一次进入页面的地址，所以IOS获取签名的url需要传第一次进入的页面url

## Code

[微信的jssdk](https://res2.wx.qq.com/open/js/jweixin-1.6.0.js)，我是把文件copy了一份放在了本地。

**jweixin-1.6.0**

```js

!function(e,n){"function"==typeof define&&(define.amd||define.cmd)?define(function(){return n(e)}):n(e,!0)}(this,function(o,e){if(!o.jWeixin){var n,c={config:"preVerifyJSAPI",onMenuShareTimeline:"menu:share:timeline",onMenuShareAppMessage:"menu:share:appmessage",onMenuShareQQ:"menu:share:qq",onMenuShareWeibo:"menu:share:weiboApp",onMenuShareQZone:"menu:share:QZone",previewImage:"imagePreview",getLocation:"geoLocation",openProductSpecificView:"openProductViewWithPid",addCard:"batchAddCard",openCard:"batchViewCard",chooseWXPay:"getBrandWCPayRequest",openEnterpriseRedPacket:"getRecevieBizHongBaoRequest",startSearchBeacons:"startMonitoringBeacons",stopSearchBeacons:"stopMonitoringBeacons",onSearchBeacons:"onBeaconsInRange",consumeAndShareCard:"consumedShareCard",openAddress:"editAddress"},a=function(){var e={};for(var n in c)e[c[n]]=n;return e}(),i=o.document,t=i.title,r=navigator.userAgent.toLowerCase(),s=navigator.platform.toLowerCase(),d=!(!s.match("mac")&&!s.match("win")),u=-1!=r.indexOf("wxdebugger"),l=-1!=r.indexOf("micromessenger"),p=-1!=r.indexOf("android"),f=-1!=r.indexOf("iphone")||-1!=r.indexOf("ipad"),m=(n=r.match(/micromessenger\/(\d+\.\d+\.\d+)/)||r.match(/micromessenger\/(\d+\.\d+)/))?n[1]:"",g={initStartTime:L(),initEndTime:0,preVerifyStartTime:0,preVerifyEndTime:0},h={version:1,appId:"",initTime:0,preVerifyTime:0,networkType:"",isPreVerifyOk:1,systemType:f?1:p?2:-1,clientVersion:m,url:encodeURIComponent(location.href)},v={},S={_completes:[]},y={state:0,data:{}};O(function(){g.initEndTime=L()});var I=!1,_=[],w={config:function(e){B("config",v=e);var t=!1!==v.check;O(function(){if(t)M(c.config,{verifyJsApiList:C(v.jsApiList),verifyOpenTagList:C(v.openTagList)},function(){S._complete=function(e){g.preVerifyEndTime=L(),y.state=1,y.data=e},S.success=function(e){h.isPreVerifyOk=0},S.fail=function(e){S._fail?S._fail(e):y.state=-1};var t=S._completes;return t.push(function(){!function(){if(!(d||u||v.debug||m<"6.0.2"||h.systemType<0)){var i=new Image;h.appId=v.appId,h.initTime=g.initEndTime-g.initStartTime,h.preVerifyTime=g.preVerifyEndTime-g.preVerifyStartTime,w.getNetworkType({isInnerInvoke:!0,success:function(e){h.networkType=e.networkType;var n="https://open.weixin.qq.com/sdk/report?v="+h.version+"&o="+h.isPreVerifyOk+"&s="+h.systemType+"&c="+h.clientVersion+"&a="+h.appId+"&n="+h.networkType+"&i="+h.initTime+"&p="+h.preVerifyTime+"&u="+h.url;i.src=n}})}}()}),S.complete=function(e){for(var n=0,i=t.length;n<i;++n)t[n]();S._completes=[]},S}()),g.preVerifyStartTime=L();else{y.state=1;for(var e=S._completes,n=0,i=e.length;n<i;++n)e[n]();S._completes=[]}}),w.invoke||(w.invoke=function(e,n,i){o.WeixinJSBridge&&WeixinJSBridge.invoke(e,x(n),i)},w.on=function(e,n){o.WeixinJSBridge&&WeixinJSBridge.on(e,n)})},ready:function(e){0!=y.state?e():(S._completes.push(e),!l&&v.debug&&e())},error:function(e){m<"6.0.2"||(-1==y.state?e(y.data):S._fail=e)},checkJsApi:function(e){M("checkJsApi",{jsApiList:C(e.jsApiList)},(e._complete=function(e){if(p){var n=e.checkResult;n&&(e.checkResult=JSON.parse(n))}e=function(e){var n=e.checkResult;for(var i in n){var t=a[i];t&&(n[t]=n[i],delete n[i])}return e}(e)},e))},onMenuShareTimeline:function(e){P(c.onMenuShareTimeline,{complete:function(){M("shareTimeline",{title:e.title||t,desc:e.title||t,img_url:e.imgUrl||"",link:e.link||location.href,type:e.type||"link",data_url:e.dataUrl||""},e)}},e)},onMenuShareAppMessage:function(n){P(c.onMenuShareAppMessage,{complete:function(e){"favorite"===e.scene?M("sendAppMessage",{title:n.title||t,desc:n.desc||"",link:n.link||location.href,img_url:n.imgUrl||"",type:n.type||"link",data_url:n.dataUrl||""}):M("sendAppMessage",{title:n.title||t,desc:n.desc||"",link:n.link||location.href,img_url:n.imgUrl||"",type:n.type||"link",data_url:n.dataUrl||""},n)}},n)},onMenuShareQQ:function(e){P(c.onMenuShareQQ,{complete:function(){M("shareQQ",{title:e.title||t,desc:e.desc||"",img_url:e.imgUrl||"",link:e.link||location.href},e)}},e)},onMenuShareWeibo:function(e){P(c.onMenuShareWeibo,{complete:function(){M("shareWeiboApp",{title:e.title||t,desc:e.desc||"",img_url:e.imgUrl||"",link:e.link||location.href},e)}},e)},onMenuShareQZone:function(e){P(c.onMenuShareQZone,{complete:function(){M("shareQZone",{title:e.title||t,desc:e.desc||"",img_url:e.imgUrl||"",link:e.link||location.href},e)}},e)},updateTimelineShareData:function(e){M("updateTimelineShareData",{title:e.title,link:e.link,imgUrl:e.imgUrl},e)},updateAppMessageShareData:function(e){M("updateAppMessageShareData",{title:e.title,desc:e.desc,link:e.link,imgUrl:e.imgUrl},e)},startRecord:function(e){M("startRecord",{},e)},stopRecord:function(e){M("stopRecord",{},e)},onVoiceRecordEnd:function(e){P("onVoiceRecordEnd",e)},playVoice:function(e){M("playVoice",{localId:e.localId},e)},pauseVoice:function(e){M("pauseVoice",{localId:e.localId},e)},stopVoice:function(e){M("stopVoice",{localId:e.localId},e)},onVoicePlayEnd:function(e){P("onVoicePlayEnd",e)},uploadVoice:function(e){M("uploadVoice",{localId:e.localId,isShowProgressTips:0==e.isShowProgressTips?0:1},e)},downloadVoice:function(e){M("downloadVoice",{serverId:e.serverId,isShowProgressTips:0==e.isShowProgressTips?0:1},e)},translateVoice:function(e){M("translateVoice",{localId:e.localId,isShowProgressTips:0==e.isShowProgressTips?0:1},e)},chooseImage:function(e){M("chooseImage",{scene:"1|2",count:e.count||9,sizeType:e.sizeType||["original","compressed"],sourceType:e.sourceType||["album","camera"]},(e._complete=function(e){if(p){var n=e.localIds;try{n&&(e.localIds=JSON.parse(n))}catch(e){}}},e))},getLocation:function(e){},previewImage:function(e){M(c.previewImage,{current:e.current,urls:e.urls},e)},uploadImage:function(e){M("uploadImage",{localId:e.localId,isShowProgressTips:0==e.isShowProgressTips?0:1},e)},downloadImage:function(e){M("downloadImage",{serverId:e.serverId,isShowProgressTips:0==e.isShowProgressTips?0:1},e)},getLocalImgData:function(e){!1===I?(I=!0,M("getLocalImgData",{localId:e.localId},(e._complete=function(e){if(I=!1,0<_.length){var n=_.shift();wx.getLocalImgData(n)}},e))):_.push(e)},getNetworkType:function(e){M("getNetworkType",{},(e._complete=function(e){e=function(e){var n=e.errMsg;e.errMsg="getNetworkType:ok";var i=e.subtype;if(delete e.subtype,i)e.networkType=i;else{var t=n.indexOf(":"),o=n.substring(t+1);switch(o){case"wifi":case"edge":case"wwan":e.networkType=o;break;default:e.errMsg="getNetworkType:fail"}}return e}(e)},e))},openLocation:function(e){M("openLocation",{latitude:e.latitude,longitude:e.longitude,name:e.name||"",address:e.address||"",scale:e.scale||28,infoUrl:e.infoUrl||""},e)},getLocation:function(e){M(c.getLocation,{type:(e=e||{}).type||"wgs84"},(e._complete=function(e){delete e.type},e))},hideOptionMenu:function(e){M("hideOptionMenu",{},e)},showOptionMenu:function(e){M("showOptionMenu",{},e)},closeWindow:function(e){M("closeWindow",{},e=e||{})},hideMenuItems:function(e){M("hideMenuItems",{menuList:e.menuList},e)},showMenuItems:function(e){M("showMenuItems",{menuList:e.menuList},e)},hideAllNonBaseMenuItem:function(e){M("hideAllNonBaseMenuItem",{},e)},showAllNonBaseMenuItem:function(e){M("showAllNonBaseMenuItem",{},e)},scanQRCode:function(e){M("scanQRCode",{needResult:(e=e||{}).needResult||0,scanType:e.scanType||["qrCode","barCode"]},(e._complete=function(e){if(f){var n=e.resultStr;if(n){var i=JSON.parse(n);e.resultStr=i&&i.scan_code&&i.scan_code.scan_result}}},e))},openAddress:function(e){M(c.openAddress,{},(e._complete=function(e){e=function(e){return e.postalCode=e.addressPostalCode,delete e.addressPostalCode,e.provinceName=e.proviceFirstStageName,delete e.proviceFirstStageName,e.cityName=e.addressCitySecondStageName,delete e.addressCitySecondStageName,e.countryName=e.addressCountiesThirdStageName,delete e.addressCountiesThirdStageName,e.detailInfo=e.addressDetailInfo,delete e.addressDetailInfo,e}(e)},e))},openProductSpecificView:function(e){M(c.openProductSpecificView,{pid:e.productId,view_type:e.viewType||0,ext_info:e.extInfo},e)},addCard:function(e){for(var n=e.cardList,i=[],t=0,o=n.length;t<o;++t){var r=n[t],a={card_id:r.cardId,card_ext:r.cardExt};i.push(a)}M(c.addCard,{card_list:i},(e._complete=function(e){var n=e.card_list;if(n){for(var i=0,t=(n=JSON.parse(n)).length;i<t;++i){var o=n[i];o.cardId=o.card_id,o.cardExt=o.card_ext,o.isSuccess=!!o.is_succ,delete o.card_id,delete o.card_ext,delete o.is_succ}e.cardList=n,delete e.card_list}},e))},chooseCard:function(e){M("chooseCard",{app_id:v.appId,location_id:e.shopId||"",sign_type:e.signType||"SHA1",card_id:e.cardId||"",card_type:e.cardType||"",card_sign:e.cardSign,time_stamp:e.timestamp+"",nonce_str:e.nonceStr},(e._complete=function(e){e.cardList=e.choose_card_info,delete e.choose_card_info},e))},openCard:function(e){for(var n=e.cardList,i=[],t=0,o=n.length;t<o;++t){var r=n[t],a={card_id:r.cardId,code:r.code};i.push(a)}M(c.openCard,{card_list:i},e)},consumeAndShareCard:function(e){M(c.consumeAndShareCard,{consumedCardId:e.cardId,consumedCode:e.code},e)},chooseWXPay:function(e){M(c.chooseWXPay,V(e),e)},openEnterpriseRedPacket:function(e){M(c.openEnterpriseRedPacket,V(e),e)},startSearchBeacons:function(e){M(c.startSearchBeacons,{ticket:e.ticket},e)},stopSearchBeacons:function(e){M(c.stopSearchBeacons,{},e)},onSearchBeacons:function(e){P(c.onSearchBeacons,e)},openEnterpriseChat:function(e){M("openEnterpriseChat",{useridlist:e.userIds,chatname:e.groupName},e)},launchMiniProgram:function(e){M("launchMiniProgram",{targetAppId:e.targetAppId,path:function(e){if("string"==typeof e&&0<e.length){var n=e.split("?")[0],i=e.split("?")[1];return n+=".html",void 0!==i?n+"?"+i:n}}(e.path),envVersion:e.envVersion},e)},openBusinessView:function(e){M("openBusinessView",{businessType:e.businessType,queryString:e.queryString||"",envVersion:e.envVersion},(e._complete=function(n){if(p){var e=n.extraData;if(e)try{n.extraData=JSON.parse(e)}catch(e){n.extraData={}}}},e))},miniProgram:{navigateBack:function(e){e=e||{},O(function(){M("invokeMiniProgramAPI",{name:"navigateBack",arg:{delta:e.delta||1}},e)})},navigateTo:function(e){O(function(){M("invokeMiniProgramAPI",{name:"navigateTo",arg:{url:e.url}},e)})},redirectTo:function(e){O(function(){M("invokeMiniProgramAPI",{name:"redirectTo",arg:{url:e.url}},e)})},switchTab:function(e){O(function(){M("invokeMiniProgramAPI",{name:"switchTab",arg:{url:e.url}},e)})},reLaunch:function(e){O(function(){M("invokeMiniProgramAPI",{name:"reLaunch",arg:{url:e.url}},e)})},postMessage:function(e){O(function(){M("invokeMiniProgramAPI",{name:"postMessage",arg:e.data||{}},e)})},getEnv:function(e){O(function(){e({miniprogram:"miniprogram"===o.__wxjs_environment})})}}},T=1,k={};return i.addEventListener("error",function(e){if(!p){var n=e.target,i=n.tagName,t=n.src;if("IMG"==i||"VIDEO"==i||"AUDIO"==i||"SOURCE"==i)if(-1!=t.indexOf("wxlocalresource://")){e.preventDefault(),e.stopPropagation();var o=n["wx-id"];if(o||(o=T++,n["wx-id"]=o),k[o])return;k[o]=!0,wx.ready(function(){wx.getLocalImgData({localId:t,success:function(e){n.src=e.localData}})})}}},!0),i.addEventListener("load",function(e){if(!p){var n=e.target,i=n.tagName;n.src;if("IMG"==i||"VIDEO"==i||"AUDIO"==i||"SOURCE"==i){var t=n["wx-id"];t&&(k[t]=!1)}}},!0),e&&(o.wx=o.jWeixin=w),w}function M(n,e,i){o.WeixinJSBridge?WeixinJSBridge.invoke(n,x(e),function(e){A(n,e,i)}):B(n,i)}function P(n,i,t){o.WeixinJSBridge?WeixinJSBridge.on(n,function(e){t&&t.trigger&&t.trigger(e),A(n,e,i)}):B(n,t||i)}function x(e){return(e=e||{}).appId=v.appId,e.verifyAppId=v.appId,e.verifySignType="sha1",e.verifyTimestamp=v.timestamp+"",e.verifyNonceStr=v.nonceStr,e.verifySignature=v.signature,e}function V(e){return{timeStamp:e.timestamp+"",nonceStr:e.nonceStr,package:e.package,paySign:e.paySign,signType:e.signType||"SHA1"}}function A(e,n,i){"openEnterpriseChat"!=e&&"openBusinessView"!==e||(n.errCode=n.err_code),delete n.err_code,delete n.err_desc,delete n.err_detail;var t=n.errMsg;t||(t=n.err_msg,delete n.err_msg,t=function(e,n){var i=e,t=a[i];t&&(i=t);var o="ok";if(n){var r=n.indexOf(":");"confirm"==(o=n.substring(r+1))&&(o="ok"),"failed"==o&&(o="fail"),-1!=o.indexOf("failed_")&&(o=o.substring(7)),-1!=o.indexOf("fail_")&&(o=o.substring(5)),"access denied"!=(o=(o=o.replace(/_/g," ")).toLowerCase())&&"no permission to execute"!=o||(o="permission denied"),"config"==i&&"function not exist"==o&&(o="ok"),""==o&&(o="fail")}return n=i+":"+o}(e,t),n.errMsg=t),(i=i||{})._complete&&(i._complete(n),delete i._complete),t=n.errMsg||"",v.debug&&!i.isInnerInvoke&&alert(JSON.stringify(n));var o=t.indexOf(":");switch(t.substring(o+1)){case"ok":i.success&&i.success(n);break;case"cancel":i.cancel&&i.cancel(n);break;default:i.fail&&i.fail(n)}i.complete&&i.complete(n)}function C(e){if(e){for(var n=0,i=e.length;n<i;++n){var t=e[n],o=c[t];o&&(e[n]=o)}return e}}function B(e,n){if(!(!v.debug||n&&n.isInnerInvoke)){var i=a[e];i&&(e=i),n&&n._complete&&delete n._complete,console.log('"'+e+'",',n||"")}}function L(){return(new Date).getTime()}function O(e){l&&(o.WeixinJSBridge?e():i.addEventListener&&i.addEventListener("WeixinJSBridgeReady",e,!1))}});

```


**jweixin.js**

```js

  /**
   * @Author: deephug
   * @Description: 基于微信sdk的封装
   * @Date: 2022-10-26 16:27:00
   */
  import '../static/js/jweixin-1.6.0'; // 引入jssdk文件
  import { getSdkSignature } from '@requests/common'; // 封装请求微信签名的方法
  class Wx {
      constructor() {
          this.signMap = new Map(); // 存储地址url认证信息sign
          // 分享默认参数
          this.defaultShareConfig = {
              title: '健康徐汇-互联网诊疗平台',
              desc: '健康徐汇打造一站式线上服务平台，为居民提供预约挂号、家庭医生、互联网医院、健康管理、数字健康档案、康复护理、妇幼保健等服务，为居民健康护航！',
              link: location.href,
              imgUrl: 'https://wx.wondersph.com/%E5%81%A5%E5%BA%B7%E5%BE%90%E6%B1%87/%E5%81%A5%E5%BA%B7%E5%BE%90%E6%B1%87logo.png',
              jsApiList: ['updateAppMessageShareData', 'updateTimelineShareData', 'scanQRCode'], // 分享给朋友分享到QQ，分享到朋友圈，扫一扫
              openTagList: ['wx-open-launch-weapp'], // 微信开放标签 必填，需要使用的JS接口列表
              globalShareLink: GLOBAL_DATA.globalShareLink, // 全局分享跳转的URL http://10.1.93.110:82/xuhuiH5/openWeiXin https://jkxh.xuhuiheart.com/xuhuiH5/openWeiXin
          };
          // 分享参数
          this.wxShareConfig = {};
      }

    /*
        deephugShare injectAndShare getShareAuth 这三个方法里面都有 return  new Promise，目的是为了在确保 wx.config 通过之后，调用一些 jssdk 其他的api
    */

      /**
       * 
       * @param {Object} config 分页配置信息，包括title，desc，等等；
       */
      deephugShare( config = {} ) {
          return new Promise(async (resolve, reject) => {
              this.wxShareConfig.title = config.title || this.defaultShareConfig.title;
              this.wxShareConfig.desc = config.desc || this.defaultShareConfig.desc;
              this.wxShareConfig.link = config.link || this.defaultShareConfig.link;
              this.wxShareConfig.imgUrl = config.imgUrl || this.defaultShareConfig.imgUrl;
              this.wxShareConfig.jsApiList = config.jsApiList || this.defaultShareConfig.jsApiList;
              this.wxShareConfig.openTagList = config.openTagList || this.defaultShareConfig.openTagList;

              const signMsg = this.signMap.has(this.wxShareConfig.link);
              if(signMsg) {
                  // 注入签名信息， 并调用分享
                  console.log('调用分享');
                  await this.injectAndShare(signMsg);
                  resolve();
              } else {
                  // 当前分享链接不存在签名 = 更新签名信息
                  // 获取签名信息
                  console.log('调用获取签名接口');
                  await this.getShareAuth(this.wxShareConfig.link);
                  resolve();
              }
          });
      }

      /**
       * 
       * @param {Object} signMsg 接口签名信息
       * @desc jssdk签名权限注入，并添加分享事件监听器
       */
      injectAndShare(signMsg) {
          return new Promise((resolve, reject) => {
              this.wx.config({
                  debug: false, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
                  appId: signMsg.appId, // 必填，公众号的唯一标识
                  timestamp: signMsg.timestamp, // 必填，生成签名的时间戳
                  nonceStr: signMsg.nonceStr, // 必填，生成签名的随机串
                  signature: signMsg.signature, // 必填，签名
                  jsApiList: ['updateAppMessageShareData', 'updateTimelineShareData', 'scanQRCode'],
                  openTagList: ['wx-open-launch-weapp']
              });

              this.wx.ready(() => {
                  resolve();
                  const { title, desc, imgUrl, } = this.wxShareConfig;
                  const { globalShareLink } = this.defaultShareConfig;
                  console.log('wx.ready', '######');
                  this.wx.updateAppMessageShareData({
                      title,desc,link: globalShareLink,imgUrl,
                      success: () => {
                          console.log('分享成功！');
                      },
                      cancel: () => {
                          console.log('分享取消！');
                      },
                      fail: () => {
                          console.log('分享失败！');
                      }
                  });
                  this.wx.updateTimelineShareData({
                      title,desc,link: globalShareLink,imgUrl,
                      success: () => {
                          console.log('分享成功 -- 朋友圈！');
                      },
                      cancel: () => {
                          console.log('分享取消 -- 朋友圈！');
                      },
                      fail: () => {
                          console.log('分享失败 -- 朋友圈！');
                      }
                  });
              });
              this.wx.error((res) => {
                  console.log('wx error:', res);
                  reject(error);
              });
          });
      }

      /**
       * 
       * @param {url} shareUrl 
       * @description 从服务器获取分享页面签名信息
       */
      getShareAuth(shareUrl) {
          return new Promise(async (resolve, reject) => {
              try {
                  console.log('待获取权限的地址', shareUrl);
                  let res = await getSdkSignature({
                      url: encodeURIComponent(window['entryUrl'] || shareUrl) // window['entryUrl'] 是给ios专门设置的属性
                  });
                  this.signMap.set(shareUrl, res); // 存储信息
                  console.log(this.signMap, '5555');
                  await this.injectAndShare(res); // 注入签名信息
                  resolve();
              } catch (error) {
                  console.log(error, 'error');
                  this.$dialog.alert('获取微信配置失败');
              }
          });
      }
  }
export default Wx;


```

**main.js**

```js

// 将 jweixin 挂载到 vue 的原型上面，方便各页面调用
import Wx from '../../common/utils/jweixin';
Vue.prototype.$jweixin = new Wx();

```

**router/index.js**

```js

  index.afterEach((to) => {
      // 由于所有的接口都需要登录后传递token，否则会报错，所以我加了一个白名单，白名单里面的路由不会走默认分享，需要在每个页面login完成后再调用一下分享
      // 白名单屏蔽的是  1、进入项目的第一个页面； 2、页面不仅用到了分享功能，还需要用到别的 jssdk 的api，比如说 扫一扫 (scanQRCode) 
      // 由于存在动态路由，屏蔽路由页面不让其分享也是专门写的以下的方法
      let noShareWhite = ['/home', '/openWeiXin', '/hosQuestionnaire/:orderId', '/homeBlank', '/agreementNursing', '/applyResult', '/nursingHome', '/MyAppointment', '/nucleicAcidNav', '/wenjuan/index'];
      if (!to.matched.some(ele => noShareWhite.includes(ele.path))) {
          const CURRENT_URL_HISTORY = location.href; //注意我的项目使用的history；
          // const CURRENT_URL_HASH = location.href.split('#')[0]; // hash模式使用这个
          const SHARE_CONFIG = {
              title: '',
              desc: '',
              link: CURRENT_URL_HISTORY, // 要分享的页面地址不允许存在#参数
              imgUrl: '', // 分享出去链接图片
          };
          if (window.__wxjs_is_wkwebview) {
              // 判断是否在 IOS 环境下，如果是则给 window 全局对象添加一个属性，用来存贮 ios 第一次进入页面的 URL，因为 ios 的 spa 应用，如果IOS的签名的路径不是第一次进入的页面，那么就一定会失败，所以这个路由第一次进入就要储存起来。
              if (window['entryUrl'] == '' || window['entryUrl'] == undefined) {
                  // 这里和jweixin.js中getShareAuth方法获取页面url中是同一个
                  window['entryUrl'] = CURRENT_URL_HISTORY;
              }
          }
          this.$jweixin.deephugShare(SHARE_CONFIG); // 调用分享
      }
  });

```

**home.vue**

home路由在分享白名单里面，不会走默认的路由，需要在页面里面调用一下。

```js

    // 接口如果不传递 token ，则会直接报错，如果当前路由有可能是进入项目的第一个页面，则需要将其路由配置到 router/index.js 的 afterEach 的 noShareWhite 里面，然后再 login 完之后调用分享
    const personInfo = storage.getUserInfo();
    if (!personInfo) {
        this.login().then((res) => {
            this.$jweixin.deephugShare({link: location.href});	// 调用分享
            this.orderId = this.$route.params.orderId;
        });
    } else {
        this.$jweixin.deephugShare({link: location.href});	// 调用分享
        this.orderId = this.$route.params.orderId;
    }

```

**scan.vue**
```js

// 页面不仅用到了分享功能，还需要用到别的 jssdk 的api，比如说 扫一扫 (scanQRCode)。
this.$jweixin.deephugShare({link: location.href}).then(res=>{
    this.wx.scanQRCode({
        needResult: 1, // 默认为0，扫描结果由微信处理，1则直接返回扫描结果，
        scanType: ['qrCode', 'barCode'], // 可以指定扫二维码还是一维码，默认二者都有
        success: res => {
            // let qrTxt = res.resultStr;
            // 扫一扫成功
        },
        fail: res => {
            // 扫一扫失败
        },
        cancel: () => {
            // 关闭扫一扫
        }
    });
});	// 调用分享

```


**OpenWeiXin.vue**

微信分享的url不能是全地址(open.weixin.qq.com)，进到页面之后拿不到code和openid，无法调用登录，所以要有一个承载的空白页面用来拼接全地址，然后再跳转过去。
OpenWeiXin.vue 就是我的空白页面。里面对 ios 和 Android 也都进行了不同的兼容处理。
1、跳转全地址后再返回到 OpenWeiXin 路由时，ios 走不进去 created ，进不去 vue 的生命周期，写了一个监听进入前台后台的方法，用来关闭从home页面返回到 OpenWeiXin 时，关闭浏览器窗口。
2、Android 可以走到 vue 的生命周期里面，在跳转全地址之前存储 session ，在 created 判断 session 里面是否有值，如果有则关闭浏览器窗口（加一个 setTimeout ，不然不会生效）。

```js

<template>
    <section class="template-area"></section>
</template>

<script>
import storage from '../../utils/storage'; // sessionStorage
import { isIos } from '../../../../common/utils/tools'; // 判断当前是否是IOS环境
import { goOpenWeixin } from '../../requests/common'; // 组装全连接（open.weixin.qq.com）
export default {
    data() {
        return {};
    },
    mounted(){
        this.$nextTick(() => {

            let targetPath = this.$tools.getParameterByName('targetPath');
            console.log(targetPath, 'targetPath--targetPath');
            // 互联网医院分享来源，需要先判断是否实名，如果实名跳转到徐汇互联网医院
            if (targetPath == 'hospitalH5') {
                switch (PUBLIC_CODE) {
                    case '2':
                        window.location.replace(goOpenWeixin('/xuhuiH5/homeBlank?sourceFrom=2'));
                        break;
                    case '3':
                        window.location.replace(goOpenWeixin('/xuhuiH5/homeBlank?sourceFrom=2'));
                        break;
                    case '4':
                        window.location.replace(goOpenWeixin('/xuhuiH5/homeBlank?sourceFrom=2'));
                        break;
                    default:
                        this.$router.replace({
                            path: '/homeBlank',
                            query: { sourceFrom: 2 }
                        });
                        break;
                }
                return;
            }
            // 当前页面只是一个中转页面，分享的 url 是没有 open.weixin 的，此页面是为了拼接全地址（open.weixin）后再跳转到对应的页面。
            // 2022-10-12 需求，在健康徐汇所有的页面分享后的地址要全部跳转到 home 页面，所以当前版本比较简单，只做了 te 和 pe 两种地址的跳转。

            // 处理 ios 兼容问题，无论使用 href、assign、 replace ， Android的表现为：从home页面物理返回之后还是回会到当前页面。 Android解决办法：跳转的时候设置session，created的时候如果session的值为true，则使用微信的api关闭当前浏览器窗口。
            // ios的表现为：home页面物理返回后不会触发生命周期。ios解决办法：需要 监听页面切换到 ‘前台’ 的事件，然后使用微信的api关闭当前浏览器窗口。

            // ios解决办法
            if (isIos()) {
                document.addEventListener('visibilitychange', this.visibilitychange, false);
            }
            // Android解决办法
            if (storage.getOnceLoadOpenWeiXin()) {
                setTimeout(() => {
                    window.WeixinJSBridge && WeixinJSBridge.call('closeWindow');
                }, 200);
                return;
            };
            let peURL = 'https://open.weixin.qq.com/connect/oauth2/authorize?appid=wx8ea774edbad0cf5b&redirect_uri=https%3A%2F%2Fjkxh.xuhuiheart.com%2FxuhuiH5%2Fhome&response_type=code&scope=snsapi_userinfo&state=310104000000#wechat_redirect';
            let teURL = 'https://open.weixin.qq.com/connect/oauth2/authorize?appid=wx7954261f30ad20b0&redirect_uri=http%3A%2F%2F10.1.93.110%3A82%2FxuhuiH5%2Fhome&response_type=code&scope=snsapi_userinfo&state=310104000000#wechat_redirect';
            // Android跳转前设置session
            storage.setOnceLoadOpenWeiXin('true');
            switch (PUBLIC_CODE) {
                case '2':
                    window.location.replace(teURL);
                    break;
                case '3':
                    window.location.replace(peURL);
                    break;
                case '4':
                    window.location.replace(peURL);
                    break;
                default:
                    console.log('组装open.weixin全地址');
                    break;
            }
        });

    },
    destroyed() {
        window.removeEventListener('visibilitychange', this.visibilitychange, false);
    },
    methods: {
        visibilitychange() {
            if (document.visibilityState == 'hidden') {
                // alert('切换到后台');
            } else if (document.visibilityState == 'visible') {
                // alert('切换到前台');
                window.WeixinJSBridge && WeixinJSBridge.call('closeWindow');
            }
        }
    },
    components: {}
};
</script>

<style lang="less" scoped>
.template-area{
    background-color: #fff;
    height: 100%;
}
</style>

```