"# deferred" 



# jQuery的deferred对象详解

## 1.什么是deferred对象？

在我们项目中两个异步请求A和B，如果B请求依赖A请求的数据dataA,那么我们就要在A请求成功之后的回调函数里面写B请求，如果再多一个C请求，而C请求依赖B请求的数据，那么A请求的回调里面有B请求，B请求的回调里面有C请求的时候，我们的代码就变成了一场灾难，这种情况一般称为回调地狱。代码如下：

 	var url = 'XXX';
	var result;
	var XHR = new XMLHttpRequest();
	XHR.open('GET', url, true);
	XHR.send();

	XHR.onreadystatechange = function() {
	    if (XHR.readyState == 4 && XHR.status == 200) {
	        result = XHR.response;
	        console.log(result);
	
	        // 伪代码
	        var url2 = 'http:xxx.yyy.com/zzz?ddd=' + result.someParams;
	        var XHR2 = new XMLHttpRequest();
	        XHR2.open('GET', url, true);
	        XHR2.send();
	        XHR2.onreadystatechange = function() {
	            ...
	        }
	    }
    }

在回调函数方面，jQuery功能非常弱。为了改变这一点，deferred对象应运而生。可以说deferred对象就是jQuery的回调函数解决方案。

## 2.ajax的链式写法

jQuery的ajax传统写法：

	　$.ajax({
	　　　　url: "test.html",
	　　　　success: function(){},
	　　　　error:function(){}
	　　});

$.ajax接受一个对象参数，这个对象包含两个方法；success成功后的回调函数，error失败后的回调函数。

在1.5版本后 $.ajax（）操作完成后返回的是deferred对象，可以进行链式操作。如：

	$.ajax({
    	url : 'test.html'
	}).success (function (data) { 
        //成功的回调
	}).error(function () {
		//失败的回调
    });
    或者：
	$.ajax({
    	url : 'test.html'
	}).done (function (data) { 
        //成功的回调
	}).fail(function () {
		//失败的回调
    });

## 3.同一操作的多种回调和多个操作的回调

deferred对象的一大好处，就是他能添加多个回调函数
   
 
	1.一个函数的多个回调
	$.ajax()
	.done(function(){ } )
	.fail(function(){ } )
	.done(function(){ } ); //第二个回调函数
	2.多个操作的指定回调
	$.when($.ajax(), $.ajax())
	.done(function(){ })    //成功的回调
	.fail(function(){ });   //失败的回调
	先执行两个请求，都成功了才会执行done()的回调函数，有一个失败就会执行fail()；
	
## 4.deferred的一些方法

#### deferred.catch()

	当Deferred对象拒绝（reject）时，调用添加的处理程序
	
#### deferred.done()

	当Deferred对象解决时，调用添加处理程序
	
#### deferred.fail()

	当Deferred对象拒绝时，调用添加处理程序
	
#### deffered.resolve()

	解决Deffered（延迟）对象，并根据给定的args参数调用任何完成回调函数（doneCallbacks）.
	
#### deferred.reject()

	拒绝Defered（延迟）对象，并根据给定的args参数调用任何失败回调函数（failCakkbacks）。
	
#### deferred.promise()

	返回一个Promise对象用来观察当某种类型的所有兴东绑定到集合，排队与否还是已经完成。
	
#### deferred.state()

	确定一个Deferred（延迟）对象的当前状态

## 5.deferred的实例

我们不仅可以在ajax操作中使用deferred，在本地的操作中我们也可以使用。不管是同步还是异步都可以使用deferred对象的各种方法，指定回调函数。

	var dtd = $.Deferred();   //新建一个deferred对象
	fn(dtd){
		var tasks = function(){
			//dosomething；
			dtd.resolve();	  //改变deferred对象的执行状态
		}
		return dtd;
	}

fn()函数返回的是deferred对象，这就可以加上链式操作了。
	
	$.when(fn(dtd))
	.done(function(){})
    .fail(function(){})
fn()函数运行完,就会自动运行done（）方法指定的回调函数。


	2.项目中的deferred片段
	var dtd = $.Deferred();
	function test(){
		if(firmId){
			dtd.resolve();
		}else{
			var company={ 
				"params": {"name":data.data.enterpriseName} 
			}
			service.getAreaByCom(company).done(function(data){
				if(data.code==0){
					firmId=data.data[0].id;
					dtd.resolve();
				}else{
					bootbox.alert(data.msg);
					dtd.reject();
				}
			})
		}
		return dtd.promise();
	}
	$.when(test()).then(function(){					
		service.getAreaByFirmId(firmId).done(function(data){
			if(data.code==0){
				areaNumber=data.data.areaNum;
				that.model.get("tplhtml").areaNum = areaNumber;
				that.sealList();
			}else{
				bootbox.alert(data.msg);
			}
		})
		
	})	
