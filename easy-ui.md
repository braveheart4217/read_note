关于 Easy-ui 的错误:
	
	Uncaught TypeError: Cannot read property 'length' of undefined
背景：使用 `easy-ui loadData` 加载本地数据的时候

介绍：`easy-ui loadData`加载的数据的 数据格式 如下：

	{"total": 0, "rows": []}
	// total 代表后面数组的长度
	// rows 代表具体要加载的数据，请注意，他是一个数组，数组，数组
解决方案：

1. 服务端传送过来的数据
	* 此种情况一般发生在 `total = 1`的时候，因为在 `total = 1` 的时候，我们就会直接将对象丢到json数据中，这时候就会导致，传过来的 rows 并不是一个数组
	* 此时只要将对象丢到一个数组或者容器里面就行了
2. 客户端自己组装的数据
	* 刚开始我就是按照服务端传送过来的数据格式进行组装，组装代码如下
	
			var row=new Array();
			row[0] = ($('#tt').datagrid('getSelected'));
			var data = {};
			data["total"] = 1;
			data["rows"] = row;
			var jsonString = JSON.stringify(data);
	* 调试的时候可以看到自己组装生成的 `jsonString` 跟服务端传过来的是一样的，但是并没有什么卵用，还是会报上面的错误
	* 然后我发现这样是可以的
			
			$('#tt').datagrid('loadData', {"total": 0, "rows": []});
	* 最后我就直接按照上面的格式组装了，然后就好了

			$('#tt').datagrid('loadData', {"total": 1, "rows": [$('#tt').datagrid('getSelected')]});

3. 总结：还是用自己写的代码靠谱