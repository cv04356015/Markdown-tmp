# 資料驅動內容預設網址

```
http://[hostname]/SASVisualAnalytics/resources/custom_table.html
```

但搜尋整個伺服器找不到 `custom_table.html` 檔案

# 建立 nginx server

install nginx [ref](https://www.godaddy.com/garage/how-to-install-and-configure-nginx-on-centos-7/)

利用 nginx 實現讀取 html 檔

```
$ vim /etc/nginx/nginx.conf
```

or

```
$ vim /etc/nginx/conf.d/default.conf
```

default.conf 內容

```
server {
  listen 8000;
  server_name 192.168.171.248;

  location / {
    root /usr/src/sas_html;
    allow all;
  }
}
```

but, nginx.conf 需要指派 default.conf 位置(預設)

將存放 html 網頁的資料夾放置於 `/usr/src/sas_html`，即可指派 html

ex.

```
http://192.168.171.248:8000/index.html
```

以上 CentOS 碰到兩個錯誤

* configuration selinux
* html 不能放置 root 使用者下(搬走即可)

configuration selinux

```
$ vim /etc/selinux/config
```

ELINUX to `disabled`

```
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of three two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

# 建立 html 檔案

```
$ vim /usr/src/sas_html/cluster_print.html
```

```
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
</head>
<body>

<div id="content1" style = "font-size:30px;text-align:center">
</div>

<script>
    
	function onMessage(evt) {
        if (evt && evt.data) {
			var results = evt.data.rowCount;
			if (results == 0) {
				document.getElementById("content1").innerHTML = "今日無新增通報或確定群聚案件";
			} else {
				document.getElementById("content1").innerHTML = "";
			}
        }
    }
	
    // Retrieve data and begin processing
	
    if (window.addEventListener) {
        // For standards-compliant web browsers
        window.addEventListener("message", onMessage, false);
    } else {
        window.attachEvent("onmessage", onMessage);
    }

</script>
</body>
</html>
```

透過 JS 可以讀取 SAS Viya 資料 

evt.data.data 即可看到所有資料，其中 evt.data.data[1] 為第一筆 evt.data.data[2] 為第一筆

event.data JSON object. Here are some of its attributes:

```
resultName
The name of the associated query result. This name is needed for communicating any messages from the data-driven content object back to SAS Visual Analytics.

data
The query results stored in a two-dimensional array. The data is in row-major order. So, event.data.data[0] is the first row of data and event.data.data[0][0] is the first column in the first row. The data in this array is unformatted for measures. Specifying a format for a measure has no impact on the data returned. Dates and datetimes are formatted, so the data reflects the format that is specified on any date or datetime variable.

rowCount
The number of rows of data returned. If all the data has been filtered out or no data items are assigned to the object, then the row count is 0.

columns
An array of column objects that let the author determine the type of data and other attributes such as format and label.

parameters
An array of parameter objects that the data-driven content object consumes when executing the query. This array of parameter objects enables the author to access the current value of the parameters and other parameter attributes. Only the parameters that are used by the query are returned in this array.
```

當有資料輸入時，onMessage 執行命令

寫法可以參考
* [Programming Considerations for Data-Driven Visualizations](http://go.documentation.sas.com/?cdcId=vacdc&cdcVersion=8.2&docsetId=varef&docsetTarget=n109mqtyl6quiun1mwfgtcn2s68b.htm&locale=us)
* [Data-Driven Content: leveraging third-party visualizations in SAS Visual Analytics (part 1 of 2)](https://communities.sas.com/t5/SAS-Communities-Library/Data-Driven-Content-leveraging-third-party-visualizations-in-SAS/ta-p/437303)
* [Data-Driven Content: leveraging third-party visualizations in SAS Visual Analytics (part 2 of 2)](https://communities.sas.com/t5/SAS-Communities-Library/Data-Driven-Content-leveraging-third-party-visualizations-in-SAS/ta-p/437352)

說明
* [Defining URL Mappings for Data-Driven Content](http://documentation.sas.com/?docsetId=vareports&docsetTarget=n0p8np8a8dcdn3n1aajk0kacgwr5.htm&docsetVersion=8.2&locale=en)
* [Working with Data Role Assignments](http://documentation.sas.com/?docsetId=vareportdata&docsetTarget=n18ob1xj9ozo0vn17ajv4yohwtpc.htm&docsetVersion=8.2&locale=en)

Example
* [BarChart and a c3 BarChart](https://github.com/sassoftware/sas-visualanalytics-thirdpartyvisualizations)
