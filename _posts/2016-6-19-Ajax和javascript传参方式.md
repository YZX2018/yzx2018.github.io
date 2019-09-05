---
layout: post
title: "Ajax和javascript传参方式"
date: 2016-06-19 
tags: java 
---

 js方法封装入参数据   把js的以json形式传参

{"sTitle":"操作","mData":"isReview","mRender":function(data,type,full){

​                        var info ={'ynBaseCustomerEnterpriseId':full.ynBaseCustomerEnterpriseId,

​                           'ynBaseCustomerEnterpriseName':full.ynBaseCustomerEnterpriseName,

​                           'ynBaseStaffCompanyId':full.ynBaseStaffCompanyId,

​                           'ynBaseStaffCompanyName':full.ynBaseStaffCompanyName,

​                           'workId':full.workId};

​                       console.log(JSON.stringify(info));

​                        if(data==""){

​    return"<button class='btn green' data-toggle='modal' data-target='#myModal' onclick='resetRaty("+JSON.stringify(info)+")'><i class='fafa-plus'></i>新增回访单</button>";

​                        }elseif(data=="1"){

​                            return '<ahref="">查看回访单</a>';

​                        }

​                    }}

\-----------------------------------------------------------------------------------------------------



传一个字符串变量

**var**a=**'<aclass="deletebtndefaultbtn-xsblack"onclick="position(****\'****'**+row.**mbMonitorInfomationId**+**'****\'****)"><iclass="fafa-level-down"></i>****定位****</a>'**;

\------------------------------------------------------------------





封装js对象转成json字符

**function***add*(){

*//debugger;*

**var**params=**new***Object*();

*//params.ynBaseStaffCompanyName=$("#ynBaseStaffCompanyId").find("option:selected").text();*

params.**ynBaseStaffCompanyName**=$(**"#ynBaseStaffCompanyName"**).val();

params.**ynBaseStaffCompanyId**=$(**"#ynBaseStaffCompanyId"**).val();

params.**workId**=$(**"#workId"**).val();

params.**ynBaseCustomerEnterpriseId**=$(**"#ynBaseCustomerEnterpriseId"**).val();

params.**ynBaseCustomerEnterpriseName**=$(**"#ynBaseCustomerEnterpriseName"**).val();

params.**mbContactsUserName**=$(**"#mbContactsUserName"**).val();

params.**mbContactsUserPhone**=$(**"#mbContactsUserPhone"**).val();

params.**mbEvaluationDescription**=$(**"#mbEvaluationDescription"**).val();

*//**暂时只有**2**种评价内容*

params.**mbCompositeScore**=(*parseFloat*($(**"#score0"**).val())**parseFloat*($(**"#mbReturnVisitScoringStandardweight0"**).val()))+(*parseFloat*($(**"#score1"**).val())**parseFloat*($(**"#mbReturnVisitScoringStandardweight1"**).val()));



**var**list=**new***Array*();

**var**obj1=**new***Object*();

obj1.**mbReturnVisitScoringStandardId**=$(**"#mbReturnVisitScoringStandardId0"**).val();

obj1.**mbReturnVisitScoringStandardTotal**=$(**"#mbReturnVisitScoringStandardTotal1"**).val();

obj1.**mbReturnVisitScoringStandardContent**=$(**"#content0"**).val();

obj1.**mbCurrentScore**=$(**"#score0"**).val();

list.push(obj1);

**var**obj2=**new***Object*();

obj2.**mbReturnVisitScoringStandardId**=$(**"#mbReturnVisitScoringStandardId1"**).val();

obj2.**mbReturnVisitScoringStandardTotal**=$(**"#mbReturnVisitScoringStandardTotal1"**).val();

obj2.**mbReturnVisitScoringStandardContent**=$(**"#content1"**).val();

obj2.**mbCurrentScore**=$(**"#score1"**).val();

list.push(obj2);

params.**list**=list;

**console**.log(*JSON*.*stringify*(params));



**if**($(**"#ynBaseStaffCompanyId"**).val()==**""**){

bootbox.alert(**"****请选择抢修单位****"**);

**returnfalse**;

}

**if**($(**"#workId"**).val()==**""**){

bootbox.alert(**"****请填写抢修单号****"**);

**returnfalse**;

}

**if**($(**"#ynBaseCustomerEnterpriseId"**).val()==**""**){

bootbox.alert(**"****请选择用电企业****"**);

**returnfalse**;

}



$.ajax({

**type**:**"post"**,

**contentType**:**"application/json"**,

**dataType**:**"json"**,

**url**:**"****${****rc**.**contextPath****}****/clientReview/add"**,

**data**:*JSON*.*stringify*(params),

success:**function**(data){

**if**(data.**status**){

alert(**"****新增成功****"**);

**window**.**location**.**href**=**"****${****rc**.**contextPath****}****/monitoringCenter/clientReviewPage/clientReview"**

}**else**{

alert(data.errorMessage);

}

}

})

}