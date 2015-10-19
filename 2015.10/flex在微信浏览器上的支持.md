### flex在微信浏览器中的使用

在微信中flex布局只能使用-webkit-box,同时不能支持自动换行

    display: -webkit-box;
    display: -webkit-flex;
    display: -ms-flexbox;
    display: flex;

---

    -webkit-box-flex: 1;
    -webkit-flex: 1;
    flex: 1;

