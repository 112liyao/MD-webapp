//lookFlow.vue
<template>
	<div class="lookFlow">
		<!-- 内容 -->
		<div class="container">
			<div class="title-box">
				<img slot="icon" src="../../static/icon/left.svg" width="24" height="24" data="0" v-on:click="toPage">
				<div class="m-office">
					查看流程
				</div>
				<div class="m-icon">
					<!-- <div class="office-icon">
						<img slot="icon" src="../../static/icon/refresh.svg" width="24" height="24" v-on:click="refresh">
					</div> -->
				</div>
			</div>
			<div class="logArea">
				<div class="flowArea">
					<h3>{{ leaveType }}</h3>
					<div class="flowTitle"><p></p>
						<div  class="flowTitle1">
							申请人
						</div>
						<div  class="flowTitle2">
							{{ userName }}
						</div>
					</div>
					<div class="flowStatus">
						<div  class="flowStatus1">
							标题
						</div>
						<div  class="flowStatus2">
							{{ title }}
						</div>
					</div>
					<div class="flowStatus" v-show="haveTime">
						<div  class="flowStatus1">
							开始时间
						</div>
						<div  class="flowStatus2">
							{{ startTime }}
						</div>
					</div>
					<div class="flowStatus" v-show="haveTime">
						<div  class="flowStatus1">
							结束时间
						</div>
						<div  class="flowStatus2">
							{{ endTime }}
						</div>
					</div>
					<div class="flowStatus">
						<div  class="flowStatus1">
							{{ because }}
						</div>
						<div  class="flowStatus2">
							{{ reason }}
						</div>
					</div>
				</div>
				<div class="flowArea">
					<h3>流程</h3>
					<div class="flowMould">
						<p>审批人 <span class="iconApprover"></span></p>
						<div class="flowChart examine">
						</div>
						<div class="flowChart mustExamine">
						</div>
					</div>
					<div class="flowMould">
						<p>抄送人 <span class="iconCoppy"></span></p>
						<div class="flowChart copyTo">
						</div>
						<div class="flowChart mustCopyTo">
						</div>
					</div>
				</div>
			</div>
		</div>
		<div class="flowBottom" v-on:click="ding">
		</div>
	</div>
</template>

<!-- script -->
<script>
import arrow from '../../static/icon/arrow.svg';
import { Toast } from 'mint-ui';
export default {
	data() {
		return {
			user:'',
			leaveType:'',
			userName:'',
			title:'',
			startTime:'',
			endTime:'',
			reason:'',
			haveTime:false,
			because:''
		}
	},
	mounted () {
		var that = this;
		var info = {
			processRecordId:this.$leaveType.processRecordId
		}
		that.$index.ajax(that,'/phMyRelated/getProcessFormMessage.ph',info,function(data){
			data = $.parseJSON(data);
			if (data.typeId == '001') {
				if (data.leaveType == '1') {
					that.leaveType = '事假';
				}else{
					that.leaveType = '病假';
				}
				that.haveTime = true;
				that.because = '请假原因';
			}else if (data.typeId == '002') {
				that.leaveType = '公出';
				that.haveTime = true;
				that.because = '公出原因';
			}else if (data.typeId == '003') {
				that.leaveType = '离职';
				that.because = '离职原因';
			}else if (data.typeId == '007') {
				that.leaveType = '补卡';
				that.because = '缺卡原因';
			}
			that.user = data;
			that.userName = data.userName;
			that.title = data.title;
			that.startTime = data.startTimeStr;
			that.endTime = data.endTimeStr;
			that.reason = data.reason;

			that.$index.ajax(that,'/phMyProcess/getExamineIdea.ph',info,function(data){git stash pop
				if (data.length > 0 ) {
					var str = '';
					for(var i=0;i<data.length;i++){
						str += '<div class="flowArea">'
					         + '<div class="flowStatus" style="border-top:none;padding:4px;">'
						     + '<div  class="flowStatus1" style="line-height:40px;">审批人</div>'
						     +'<div  class="flowStatus2">'
							 +'<span class="head" style="background:'+that.changeColor()+'">'+(data[i].executorName|| "")+'</span>'
							 + (data[i].executorName|| "")
							 +'</div>'
							 +'</div>'
							 +'<div class="flowStatus">'
						     +'<div  class="flowStatus1">审批意见</div>'
						     +'<div  class="flowStatus2">'+ (data[i].executorIdea || "") +'</div></div>';
						if(data[i].examineStatus == 1){
							str += '<div class="button examine-btn">待审</div>';
						}else if(data[i].examineStatus == 2){
							str += '<div class="button">通过</div>';
						}else if(data[i].examineStatus == 3){
							str += '<div class="button reject-btn">驳回</div>';
						}else if(data[i].examineStatus == 4){
							str += '<div class="button reject-btn">撤回</div>';
						}
						str += '</div>';
					}
					$('.lookFlow .logArea').append(str);
					var status = data[data.length-1].examineStatus;
					if (status == '1') {
						$('.lookFlow .flowBottom').attr('status','1');
						if(that.user.userId == that.$user.userId){
							$('.lookFlow .flowBottom').show();
							$('.lookFlow .flowBottom').text('催办');
						}
					}else{
						$('.lookFlow .flowBottom').show();
						if(status == '2'){
							$('.lookFlow .flowBottom').attr('status','2');
							$('.lookFlow .flowBottom').text('已通过');
						}else if(status == '3'){
							$('.lookFlow .flowBottom').attr('status','3');
							$('.lookFlow .flowBottom').addClass('reject-flowBottom');
							$('.lookFlow .flowBottom').text('已被驳回');
						}else if(status == '4'){
							$('.lookFlow .flowBottom').attr('status','4');
							$('.lookFlow .flowBottom').addClass('reject-flowBottom');
							$('.lookFlow .flowBottom').text('已被撤回');
						}
					}
				}
				/*else{
					$('.lookFlow .flowBottom').attr('status','1');
					$('.lookFlow .flowBottom').text('催办');
				}*/
			});
		});
		that.$index.ajax(that,'/phMyRelated/getProcessExecutor.ph',info,function(data){
			//data = $.parseJSON(data);
			if (data.excuteUserHead) {
				var excuteUserHead = data.excuteUserHead;
				var str = '';
				for (var i = 0; i<excuteUserHead.length ; i++) {
				 	str += '<span class="flowPeople" id="'+excuteUserHead[i].executorId+'" taskId="'+excuteUserHead[i].taskId+'"><span class="head" style="background:'+that.changeColor()+'">'+excuteUserHead[i].executorName+'</span><br/>'+excuteUserHead[i].executorName+'</span>';
				 	str += '<img slot="icon" src="'+arrow+'" width="24" height="24" >';
				};
				$('.lookFlow .flowArea .examine').html(str);
				$('.lookFlow .flowArea .examine img:last').remove();
			}
			if(data.mustExcuteUserHead){
				var mustExcuteUserHead = data.mustExcuteUserHead;
				var mustExStr = '';
				for(var i = 0;i<mustExcuteUserHead.length;i++){
					mustExStr += '<span class="flowPeople" id="'+mustExcuteUserHead[i].executorId+'" taskId="'+mustExcuteUserHead[i].taskId+'"><span class="head" style="background:'+that.changeColor()+'">'+mustExcuteUserHead[i].executorName+'</span><br/>'+mustExcuteUserHead[i].executorName+'</span>';
				 	mustExStr += '<img slot="icon" src="'+arrow+'" width="24" height="24" >';
				}
				$('.lookFlow .flowArea .mustExamine').html(mustExStr);
				$('.lookFlow .flowArea .mustExamine img:last').remove();
			}
			if(data.copyToUsers){
				var copyToUsers = data.copyToUsers;
				var str = '';
				for(var i = 0 ; i < copyToUsers.length ; i++){
					str += '<span class="flowPeople" id="'+copyToUsers[i].executorId+'" taskId="'+copyToUsers[i].taskId+'"><span class="head" style="background:'+that.changeColor()+'">'+copyToUsers[i].executorName+'</span><br/>'+copyToUsers[i].executorName+'</span>';
				}
				$('.lookFlow .flowArea .copyTo').html(str);
			}
			if(data.mustCopyToUsers){
				var mustCopyToUsers = data.mustCopyToUsers;
				var mustStr = '';
				for(var i = 0 ; i < mustCopyToUsers.length ; i++){
					mustStr += '<span class="flowPeople" id="'+mustCopyToUsers[i].executorId+'" taskId="'+mustCopyToUsers[i].taskId+'"><span class="head" style="background:'+that.changeColor()+'">'+mustCopyToUsers[i].executorName+'</span><br/>'+mustCopyToUsers[i].executorName+'</span>';
				}
				$('.lookFlow .flowArea .mustCopyTo').html(mustStr);
			}
		})
	},
	methods: {
		toPage:function(event){
			var el = event.currentTarget;
			var a = $(el).attr('data');
			if (a == 0) {
				this.$router.push({path:'/myFlow'});
			}
		},
		// refresh:function(){
		// 	window.location.reload()
		// },
		ding:function(event){
			var ele = event.currentTarget;
			var status = $(ele).attr('status');
			var el = this;
			if(status == '1'){
				var userName = el.items[0].executorName;//接收人
				var userId = el.items[0].executorId;
				var sendUserId = el.user.userId;
				var sendUserName = el.user.userName;//发起人
				var type = el.user.typeId;
				var whatProcess = '';
				if(type == '001'){
					whatProcess = '请假申请'
				}else if(type == '002'){
					whatProcess = '公出申请'
				}else if(type == '003'){
					whatProcess = '离职申请'
				}else if(type == '004'){
					whatProcess = '福利发放申请'
				}else if(type == '005'){
					whatProcess = '合同交单申请'
				}else if(type == '006'){
					whatProcess = '公文发放申请'
				}else if(type == '007'){
					whatProcess = '补卡申请'
				}
				var message = userName + ',' + sendUserName + '希望您尽快对他的' + whatProcess + '进行审批！谢谢！';
				el.$index.ajax(this,'/phUrgeMessage/urgeProcess.ph',{
					userId:userId,
					sendUserId:sendUserId,
					sendUserName:sendUserName,
					message:message
				},function(data){
					Toast({
						message: '催办成功！',
						position: 'center',
						duration: 2000
					});
				});
			}
		}
	}
}

</script>
