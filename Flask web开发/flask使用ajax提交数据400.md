```
<meta name="csrf-token" content="{{ csrf_token() }}">
```

```
<script>
    var csrftoken = $('meta[name=csrf-token]').attr('content');

    $.ajaxSetup({
        beforeSend: function(xhr, settings) {
            if (!/^(GET|HEAD|OPTIONS|TRACE)$/i.test(settings.type) && !this.crossDomain) {
                xhr.setRequestHeader("X-CSRFToken", csrftoken)
            }
        }
    })
</script>
```

当使用ajax方式post数据时,将csrf_token 写入请求头,

后台获取请求头后判断是否一致