# js基础知识笔记

> js
```java
<template>
    <view class="content" :style="{width: width + 'px',height:height + 'px'}">
		<live-pusher 
			class="livePusher"
			@error="error"
			@statechange="statechange" 
			:style="{width: width + 'px',height:height + 'px'}" 
			ref="livePusher" 
			id="livePusher" 
			mode="FHD"
			orientation="vertical"
			enable-camera muted>
		</live-pusher>
		
		<view class="pupop-box" v-if="showCoverView" :style="{ backgroundColor: searchInfoData.allowPass?'#008c00':'red'}">
			<view class="title-view">
				<text class="title-box">{{searchInfoData.allowPass?'请 通 行':'禁 止 通 行'}}</text>
			</view>
			<view class="result-box-jkm">
				<text style="margin: 10px;font-size: 36px;">{{searchInfoData.healthCode.statusDesc}}</text>
				<view style="border-radius: 50%;overflow: hidden;align-items: center;justify-content: center;">
					<img :src="faceImg" alt="#" style="width: 100px;height: 100px;margin-bottom: 10px;">
				</view>
				<text style="font-size: 36px;margin-bottom: 10px;">{{searchInfoData.name}}</text>
				<text style="font-size: 36px;">{{searchInfoData.idNumber}}</text>
			</view>
			<view class="box-result">
				<view class="result-box-content">
					<text style="font-size: 26px;">核 酸</text>
					<text style="margin: 20px 0;">{{searchInfoData.nuclein.resultDesc}}</text>
					<text>{{searchInfoData.nuclein.time}}</text>
				</view>
				<view class="result-box-content1">
					<text style="font-size: 26px;">疫 苗</text>
					<text style="margin: 20px 0;">{{searchInfoData.vaccin.resultDesc}}</text>
					<text>{{searchInfoData.vaccin.time||'-'}}</text>
				</view>
			</view>
			<!-- <view class="result-box-jkm">
				<text style="margin: 10px;font-size: 36px;">抗原</text>
				<text style="font-size: 26px;">{{searchInfoData.antigen.resultDesc}}</text>
				<text style="font-size: 26px;">{{searchInfoData.antigen.time||'-'}}</text>
			</view> -->
		</view>
    </view>
</template>

<script>
	import { remodeling,encryptionName,encryptionIdCard,encryptionPhone,validTime } from '@/utils/index.js';
	import $api from '@/http/api/index.js';
	import sm4 from'@/utils/SM4Encode.js'
	import { pathToBase64 } from '@/js_sdk/mmmm-image-tools/index.js';
	import { openSqlite,isOpen,addUserInformation,userInfoSQL,executeSql,closeSQL,selectInformationType,getTable,isTable } from '@/utils/sqlite.js';
	const CameraModule = uni.requireNativePlugin('CameraModule');
	const TemperatureService = uni.requireNativePlugin('TemperatureService');
	const ReadIdCardDemo = uni.requireNativePlugin('ReadIdCardDemo');
	const TTSSpeaker = uni.requireNativePlugin('Karma617-TTSSpeaker');
	const ScanQrModule = uni.requireNativePlugin('ScanQrModule');
    export default {
        data() {
			return {
				readIng: false,
				readScaning: false,
				width:'',
				height:'',
				
				livePusher: null,
				showCoverView:false,
				noData:'', // 读身份证的号码
				context: null,
				faceImg:'',
				idNo:'', // 上次查询的身份证
				readScanData:'', // 扫描到的二维码数据
				searchInfoData:{ // 接口数据
					allowPass: '', //是否允许通行
					antigen: { // 抗原信息
						effectiveTime: '', // 抗原有效期（小时）
						result: '', // 抗原结果值:1: 正常 -1：异常 0：暂无数据
						resultDesc: "", // 抗原结果描述:1：阴性、无效结果 -1：阳性、阴性(超期)、阳性(超期) 0：暂无数据
						status: '', // 抗原状态值: 1: 阴性 -1: 阳性 -2: 无效结果 0: 暂无数据
						statusDesc: "", // 抗原状态描述: 阴性、阳性、无效结果、暂无数据
						time: "" // 抗原检测时间: 格式:yyyy-MM-dd HH:mm:ss
					},
					extraInfo: "", // 附加显示信息 不是必须显示的字段，仅在特定设备、特定情况下显示。为空时不显示。不为空时，将字符串显示在界面指定位置
					healthCode: { // 健康码信息
						result: '', // 健康码结果值:1: 正常 -1：异常 0：暂无数据
						resultDesc: "", // 疫苗结果描述:正常、异常、暂无数据
						status: '',// 健康码状态值:0：暂无数据 1：绿码 2：黄码 3：红码
						statusDesc: "" // 健康码状态描述:暂无数据、绿码、黄码、红码
					},
					idNumber: "", // 身份证号
					name: "", // 姓名
					nuclein: { // 核酸信息
						effectiveTime: '', // 核酸有效期（小时）
						result: '', // 核酸结果值：1：阴性 -1：异常 0：暂无数据
						resultDesc: "",// 核酸结果描述：1：阴性 -1：阳性、阴性(超期)、阳性(超期)、待复核 0：暂无数据
						status: '', // 核酸状态值:1: 阴性 -1: 阳性 -2: 待复核 0: 暂无数据
						statusDesc: "", //阴性、阳性、待复核、暂无数据
						time: "" // 核酸采样时间:格式:yyyy-MM-dd HH:mm:ss
					},
					temperature: '', // 暂无数据的情况下，值为0。四舍五入后保留一位小数的温度
					vaccin: { // 疫苗信息
						result: '', // 疫苗结果：1：正常 -1：异常 0: 暂无数据
						resultDesc: "", // 疫苗结果描述:正常、异常、暂无数据
						times: '' // 接种剂次
					}
				}
			}
        },
		onShow() {
			this.noData = '';
			this.readScanData = '';
			this.idNo = '';
		},
		onLoad() {
			
			let res = uni.getSystemInfoSync();
			console.log(res)
			this.width = res.windowWidth;
			this.height = res.windowHeight;
			TemperatureService.connectService();
			ReadIdCardDemo.initIdCard();
			//85T1-116L-Z364-NGFQ // 面板机
			// 85T1-116L-Z3AQ-G1V8
			// 85T1-116L-Z3B8-C574
			//85T1-116L-Z374-9VEP
			// CameraModule.activeOnline({"ACTIVE_KEY":'85T1-116L-Z364-NGFQ'},(res) => {
			// 	console.log(res)
			// })
		},
        onReady() {
			// // 注意：需要在onReady中 或 onLoad 延时
			this.livePusher = uni.createLivePusherContext('livePusher', this);
			
			setTimeout(() => {
				// this.livePusher.switchCamera()
				this.startPreview();// 开启摄像头
			},500)
			TTSSpeaker.init(function(result) {
				console.log(result);
				// 返回设置的语速和音调
				TTSSpeaker.setVoiceSpeed(2);
				TTSSpeaker.setVoicePicth(1);
			});
			setInterval(() => {
				this.ReadIdCard();
				this.readScanQrModule()
			},1500)
        },
        methods: {
			error(e) {
				console.log(e)
			},
			statechange(res) {
				console.log(res)
			},
			startPreview() {
				this.livePusher.startPreview({
					success: (a) => {
						console.log("livePusher.startPreview:" + JSON.stringify(a));
					}
				});
			},
			// 获取健康码信息 身份证 姓名
			getUserInfo(idNo,name) {
				if(!idNo||!name) return uni.showToast({  title: '信息错误', icon:"error", duration: 2000 });
				this.readIng = true;
				this.searchInfoData = {
					allowPass: '', //是否允许通行
					antigen: { // 抗原信息
						effectiveTime: '', // 抗原有效期（小时）
						result: '', // 抗原结果值:1: 正常 -1：异常 0：暂无数据
						resultDesc: "", // 抗原结果描述:1：阴性、无效结果 -1：阳性、阴性(超期)、阳性(超期) 0：暂无数据
						status: '', // 抗原状态值: 1: 阴性 -1: 阳性 -2: 无效结果 0: 暂无数据
						statusDesc: "", // 抗原状态描述: 阴性、阳性、无效结果、暂无数据
						time: "" // 抗原检测时间: 格式:yyyy-MM-dd HH:mm:ss
					},
					extraInfo: "", // 附加显示信息 不是必须显示的字段，仅在特定设备、特定情况下显示。为空时不显示。不为空时，将字符串显示在界面指定位置
					healthCode: { // 健康码信息
						result: '', // 健康码结果值:1: 正常 -1：异常 0：暂无数据
						resultDesc: "", // 疫苗结果描述:正常、异常、暂无数据
						status: '',// 健康码状态值:0：暂无数据 1：绿码 2：黄码 3：红码
						statusDesc: "" // 健康码状态描述:暂无数据、绿码、黄码、红码
					},
					idNumber: "", // 身份证号
					name: "", // 姓名
					nuclein: { // 核酸信息
						effectiveTime: '', // 核酸有效期（小时）
						result: '', // 核酸结果值：1：阴性 -1：异常 0：暂无数据
						resultDesc: "",// 核酸结果描述：1：阴性 -1：阳性、阴性(超期)、阳性(超期)、待复核 0：暂无数据
						status: '', // 核酸状态值:1: 阴性 -1: 阳性 -2: 待复核 0: 暂无数据
						statusDesc: "", //阴性、阳性、待复核、暂无数据
						time: "" // 核酸采样时间:格式:yyyy-MM-dd HH:mm:ss
					},
					temperature: '', // 暂无数据的情况下，值为0。四舍五入后保留一位小数的温度
					vaccin: { // 疫苗信息
						result: '', // 疫苗结果：1：正常 -1：异常 0: 暂无数据
						resultDesc: "", // 疫苗结果描述:正常、异常、暂无数据
						times: '' // 接种剂次
					}
				};
				let params = {
					name,
					idNumber:idNo,
					deviceId:'1210980T02801275',
					temperature:0,
				}
				let val = JSON.stringify(params)
				const str = sm4.encrypt_cbc(val,'BbsHTljZ8pooAuRe','eQtGUAxXjupBACYw')
				let t = new Date().getTime();
				$api.searchByname(str)
					.then(res => {
						let ti = new Date().getTime();
						console.log("ti:"+ti)
						console.log('耗时：'+(ti-t))
						console.log(res,'获取健康码信息 身份证 姓名');
						if(res.data.success && res.data.result.code == 200) {
							let info = res.data.result.data;
							let encryptVal = sm4.decrypt_cbc(info,'BbsHTljZ8pooAuRe','eQtGUAxXjupBACYw');
							console.log('encryptVal:',JSON.parse(encryptVal))
							this.searchInfoData = JSON.parse(encryptVal);
							if(this.searchInfoData.allowPass) {
								// console.log(_this.searchInfoData.nuclein.effectiveTime+'小时核酸'+_this.searchInfoData.nuclein.resultDesc)
								this.TemperatureStr += this.searchInfoData.nuclein.effectiveTime+'小时核酸'+this.searchInfoData.nuclein.resultDesc;
								TTSSpeaker.speak(this.TemperatureStr,1);
							}else{
								TTSSpeaker.speak(res.data.result.message,1)
							}
							let time2 = setTimeout(() => {
								this.noData = '';
								this.TemperatureStr = '';
								clearTimeout(time2)
							},2000)
							if(this.idNo === this.searchInfoData.idNumber) return ;
							this.idNo = this.searchInfoData.idNumber;
							let passStatus = this.searchInfoData.allowPass ? '允许通行' : '禁止通行';
							let tiem1 = setTimeout(() => {
								this.showCoverView = true;
								let tiem3 = setTimeout(() => {
									this.showCoverView = false;
									this.readIng = false;
									clearTimeout(tiem3)
									clearTimeout(tiem1);
								},3000)
							},500)
							this.upSearchInfo({
								allowPass: passStatus,
								name:this.searchInfoData.name,
								idNumber:this.searchInfoData.idNumber,
								temperature:this.searchInfoData.temperature,
								healthCodeResult:this.searchInfoData.healthCode.result,
								healthCodeResultDesc:this.searchInfoData.healthCode.resultDesc,
								healthCodeStatus:this.searchInfoData.healthCode.status,
								healthCodeStatusDesc:this.searchInfoData.healthCode.statusDesc,
								nucleinEffectiveTime:this.searchInfoData.nuclein.effectiveTime,
								nucleinResult:this.searchInfoData.nuclein.result,
								nucleinResultDesc:this.searchInfoData.nuclein.resultDesc,
								nucleinStatus:this.searchInfoData.nuclein.status,
								nucleinStatusDesc:this.searchInfoData.nuclein.statusDesc,
								nucleinTime:this.searchInfoData.nuclein.time,
								vaccinResult:this.searchInfoData.vaccin.result,
								vaccinResultDesc:this.searchInfoData.vaccin.resultDesc,
								vaccinTimes:this.searchInfoData.vaccin.times,
								antigenEffectiveTime:this.searchInfoData.antigen.effectiveTime,
								antigenResult:this.searchInfoData.antigen.result,
								antigenResultDesc:this.searchInfoData.antigen.resultDesc,
								antigenStatus:this.searchInfoData.antigen.status,
								antigenStatusDesc:this.searchInfoData.antigen.statusDesc,
								antigenTime:this.searchInfoData.antigen.time,
								deviceId: 'test mbj',
								type:'身份证姓名',
								// operator:this.account,
								faceImg: this.faceImg
							})
						}else{
							uni.showToast({  title: '获取异常', icon:"error", duration: 2000 });
							this.noData = '';
							this.TemperatureStr = '';
							this.readIng = false;
						}
					})
					.catch(err => {
						console.log(err);
						uni.hideLoading();
						uni.showToast({  title: '接口异常', icon:"error", duration: 2000 });
						this.noData = '';
						this.TemperatureStr = '';
						this.readIng = false;
					})
			},
			// 获取健康码信息 扫码
			getQrCode(qrcode) {
				this.readScaning = true;
				let _this = this;
				_this.searchInfoData = {
					allowPass: '', //是否允许通行
					antigen: { // 抗原信息
						effectiveTime: '', // 抗原有效期（小时）
						result: '', // 抗原结果值:1: 正常 -1：异常 0：暂无数据
						resultDesc: "", // 抗原结果描述:1：阴性、无效结果 -1：阳性、阴性(超期)、阳性(超期) 0：暂无数据
						status: '', // 抗原状态值: 1: 阴性 -1: 阳性 -2: 无效结果 0: 暂无数据
						statusDesc: "", // 抗原状态描述: 阴性、阳性、无效结果、暂无数据
						time: "" // 抗原检测时间: 格式:yyyy-MM-dd HH:mm:ss
					},
					extraInfo: "", // 附加显示信息 不是必须显示的字段，仅在特定设备、特定情况下显示。为空时不显示。不为空时，将字符串显示在界面指定位置
					healthCode: { // 健康码信息
						result: '', // 健康码结果值:1: 正常 -1：异常 0：暂无数据
						resultDesc: "", // 疫苗结果描述:正常、异常、暂无数据
						status: '',// 健康码状态值:0：暂无数据 1：绿码 2：黄码 3：红码
						statusDesc: "" // 健康码状态描述:暂无数据、绿码、黄码、红码
					},
					idNumber: "", // 身份证号
					name: "", // 姓名
					nuclein: { // 核酸信息
						effectiveTime: '', // 核酸有效期（小时）
						result: '', // 核酸结果值：1：阴性 -1：异常 0：暂无数据
						resultDesc: "",// 核酸结果描述：1：阴性 -1：阳性、阴性(超期)、阳性(超期)、待复核 0：暂无数据
						status: '', // 核酸状态值:1: 阴性 -1: 阳性 -2: 待复核 0: 暂无数据
						statusDesc: "", //阴性、阳性、待复核、暂无数据
						time: "" // 核酸采样时间:格式:yyyy-MM-dd HH:mm:ss
					},
					temperature: '', // 暂无数据的情况下，值为0。四舍五入后保留一位小数的温度
					vaccin: { // 疫苗信息
						result: '', // 疫苗结果：1：正常 -1：异常 0: 暂无数据
						resultDesc: "", // 疫苗结果描述:正常、异常、暂无数据
						times: '' // 接种剂次
					}
				};
				let params = {
					qrcode,
					vendorId:'haian',
					model:'TPS980P',
					deviceId:'1210980T02801275',
					temperature:0,
				}
				let val = JSON.stringify(params)
				const str = sm4.encrypt_cbc(val,'BbsHTljZ8pooAuRe','eQtGUAxXjupBACYw');
				let t = new Date().getTime();
				console.log("t:"+t)
				$api.searchBySanc(str)
					.then(res => {
						let ti = new Date().getTime();
						console.log("ti:"+ti)
						console.log('耗时：'+(ti-t))
						console.log(res,'获取健康码信息 扫码');
						uni.hideLoading();
						if(res.data.success && res.data.result.code == 200) {
							let info = res.data.result.data;
							let encryptVal = sm4.decrypt_cbc(info,'BbsHTljZ8pooAuRe','eQtGUAxXjupBACYw');
							console.log('encryptVal:',JSON.parse(encryptVal))
							_this.searchInfoData = JSON.parse(encryptVal);
							if(_this.searchInfoData.allowPass) {
								_this.TemperatureStr += _this.searchInfoData.nuclein.effectiveTime+'小时核酸'+_this.searchInfoData.nuclein.resultDesc;
								TTSSpeaker.speak(_this.TemperatureStr,1);
							}else{
								TTSSpeaker.speak(res.data.result.message,1)
							}
							
							let time2 = setTimeout(() => {
								_this.readScanData = '';
								_this.TemperatureStr = '';
								clearTimeout(time2)
							},2500)
							let tiem1 = setTimeout(() => {
								_this.showCoverView = true;
								let tiem3 = setTimeout(() => {
									_this.showCoverView = false;
									_this.readScaning = false;
									clearTimeout(tiem3);
									clearTimeout(tiem1);
								},3000)
							},500)
							if(_this.idNo === _this.searchInfoData.idNumber) return ;
							let passStatus = _this.searchInfoData.allowPass ? '允许通行' : '禁止通行';
							_this.idNo = _this.searchInfoData.idNumber;
							_this.upSearchInfo({
								allowPass: passStatus,
								name:_this.searchInfoData.name,
								idNumber:_this.searchInfoData.idNumber,
								temperature:_this.searchInfoData.temperature,
								healthCodeResult:_this.searchInfoData.healthCode.result,
								healthCodeResultDesc:_this.searchInfoData.healthCode.resultDesc,
								healthCodeStatus:_this.searchInfoData.healthCode.status,
								healthCodeStatusDesc:_this.searchInfoData.healthCode.statusDesc,
								nucleinEffectiveTime:_this.searchInfoData.nuclein.effectiveTime,
								nucleinResult:_this.searchInfoData.nuclein.result,
								nucleinResultDesc:_this.searchInfoData.nuclein.resultDesc,
								nucleinStatus:_this.searchInfoData.nuclein.status,
								nucleinStatusDesc:_this.searchInfoData.nuclein.statusDesc,
								nucleinTime:_this.searchInfoData.nuclein.time,
								vaccinResult:_this.searchInfoData.vaccin.result,
								vaccinResultDesc:_this.searchInfoData.vaccin.resultDesc,
								vaccinTimes:_this.searchInfoData.vaccin.times,
								antigenEffectiveTime:_this.searchInfoData.antigen.effectiveTime,
								antigenResult:_this.searchInfoData.antigen.result,
								antigenResultDesc:_this.searchInfoData.antigen.resultDesc,
								antigenStatus:_this.searchInfoData.antigen.status,
								antigenStatusDesc:_this.searchInfoData.antigen.statusDesc,
								antigenTime:_this.searchInfoData.antigen.time,
								deviceId: 'test mbj',
								type:'随申码',
								// operator:_this.account,
								faceImg: _this.faceImg
							})
						}else{
							uni.showToast({  title: '获取异常', icon:"error", duration: 2000 });
							_this.readScanData = '';
							_this.TemperatureStr = '';
							_this.readScaning = false;
						}
						
					})
					.catch(err => {
						_this.readScaning = false;
						console.log(err);
						uni.hideLoading();
						uni.showToast({  title: '查询错误', icon:"error", duration: 2000 });
						_this.readScanData = '';
						_this.TemperatureStr = '';
					})
				
			},
			// 上传查询记录
			upSearchInfo(params) {
				let _this = this;
				if(!params) return ;
				let paramsStr = JSON.parse(JSON.stringify(params));
				let controls  = remodeling(paramsStr,'faceImg');
				$api._upSearchInfo(controls)
					.then(res => {
						console.log(res.data);
						_this.idNo = '';
					})
					.catch(err => {
						console.log(err)
					})
			},
			// 读身份证
			ReadIdCard() {
				let _this = this;
				ReadIdCardDemo.onGetInfo((res) => {
					// _this.ReadIdCard();
					// _this.ReadIdCardDemoRes = JSON.stringify(res)
					// base64 no apartment name period//日期 sex born  "period": "2020.01.22 - 2030.01.22", "sex": "男 / M", "born": "2001.07.19"
					console.log(res);
					let noData = res.data.no;
					console.log(noData !== _this.noData)
					if(noData !== _this.noData) {
						_this.snapshot(); // 拍照
						_this.noData = noData;
					}
					if(!this.readIng) _this.Temperature(1,{no:res.data.no,name:res.data.name});
				})
			},
			// 测温
			Temperature(type,val) {
				console.log('测温')
				let _this = this;
				TemperatureService.getTem((res) => {
					_this.TemperatureStr = res.data.temperature.toFixed(1)+'度'
					if(type == 1) {
						_this.getUserInfo(val.no,val.name)
					}
					if(type == 2) {
						_this.getQrCode(val)
					}
					// console.log(res)
				});
			},
			// 读二维码
			readScanQrModule() {
				let _this = this;
				ScanQrModule.initScan((res) => {
					console.log(res);
					let data = res.data.substring(5);
					console.log(data);
					console.log(data !== _this.readScanData);
					if(data !== _this.readScanData) {
						_this.snapshot(); // 拍照
						_this.readScanData = data;
					}
					console.log(_this.readScanData)
					if(!_this.readScaning) _this.Temperature(2,data);;
				})
			},
			// 快照
			snapshot() {
				let _this = this;
				this.livePusher.snapshot({
					success(res) {
						console.log(res)
						let tempImagePath = res.message.tempImagePath;
						console.log(tempImagePath)
						uni.compressImage({
						  src: tempImagePath,
						  quality: 50,
						  success: res => {
						    console.log(res);
							pathToBase64(res.tempFilePath)
							  .then(base64 => {
								_this.faceImg = base64;
								_this.clearCache();
								console.log(base64.split(',')[0])
								// CameraModule.faceDetection({base64:base64.split(',')[1]},(info) => {
								// 	console.log(info)
								// })
							  })
							  .catch(error => {
							    console.error(error)
							  })
						  }
						})
					}
				})
			},
			clearCache() {  
			                let that = this;  
			                let os = plus.os.name;  
			                if (os == 'Android') {  
			                    let main = plus.android.runtimeMainActivity();  
			                    let sdRoot = main.getCacheDir();  
			                    let files = plus.android.invoke(sdRoot, "listFiles");  
			                    let len = files.length;  
			                    for (let i = 0; i < len; i++) {  
			                        let filePath = '' + files[i]; // 没有找到合适的方法获取路径，这样写可以转成文件路径  
			                        plus.io.resolveLocalFileSystemURL(filePath, function(entry) {  
			                            if (entry.isDirectory) {  
			                                entry.removeRecursively(function(entry) { //递归删除其下的所有文件及子目录  
			                                    // uni.showToast({  
			                                    //     title: '缓存清理完成',  
			                                    //     duration: 2000  
			                                    // });  
			                                    that.formatSize(); // 重新计算缓存  
			                                }, function(e) {  
			                                    console.log(e.message)  
			                                });  
			                            } else {  
			                                entry.remove();  
			                            }  
			                        }, function(e) {  
			                            console.log('文件路径读取失败')  
			                        });  
			                    }  
			                } else { // ios  
			                    plus.cache.clear(function() {  
			                        uni.showToast({  
			                            title: '缓存清理完成',  
			                            duration: 2000  
			                        });  
			                        that.formatSize();  
			                    });  
			                }  
			            },
			// 打开数据库
			async openDBSql(faceId,base64,tag) {
				let _this = this;
				let faceInfo = '';
				if(isOpen()) {
					console.log('open....');
					faceInfo = await isTable('pop_face','faceInfo');
					console.log(faceInfo);
				}else{
					console.log('no open')
					const openData = await openSqlite();
					console.log(openData)
					faceInfo = await isTable('pop_face','faceInfo');
				}
				if(!faceInfo) {
					await userInfoSQL();
				}
				
				const addInfo = await addUserInformation({ loca_id:faceId, imgPath:'', imgBaes64:base64, tag });
				console.log('addInfo',addInfo)
			},
        }
    }
</script>

<style scoped>
	.content{
		/* height: 100%; */
		/* width: 100vw;
		height: 100vh; */
		align-items: center;
		justify-content: center;
		
	}
	.livePusher{
		/* width: 535px; */
		/* height: 805px; */
		/* width: 535px;
		height: 805px; */
		/* height: 800vh; */
		/* width: 800vw; */
		/* flex: 1; */
		/* width: 100%; */
		/* height: 100%; */
	}
	.pupop-box{
		align-items: center;
		position: absolute;
		left: 0;
		right: 0;
		bottom: 0;
		top: 0;
	}
	.title-view{
		padding: 30px 0;
		margin-top: 100px;
	}
	.title-box{
		font-size: 52px;
		text-align: center;
		color: #ffffff;
		font-weight: bold;
	}
	.result-box-jkm{
		align-items: center;
		background-color: #ffffff;
		border-radius: 5px;
		padding: 20px;
		width: 500px;
		height: 250px;
	}
	.box-result{
		flex-direction: row;
		align-items: flex-start;
		border-radius: 5px;
		padding: 20px;
		width: 500px;
		height: 230px;
	}
	.result-box-content{
		align-items: center;
		background-color: #ffffff;
		border-radius: 5px;
		padding: 20px;
		width: 230px;
		height: 200px;
		margin-right: 40px;
		margin-left: -20px;
	}
	.result-box-content1{
		align-items: center;
		background-color: #ffffff;
		border-radius: 5px;
		padding: 20px;
		width: 230px;
		height: 200px;
	}
</style>

```
