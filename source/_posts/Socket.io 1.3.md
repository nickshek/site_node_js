---
title: Socket.io 1.3.5 cdn 同源問題
date: 2015-04-15
categories:
    - socket.io
---
假設己經有一個HTTPS加密網站，為了令網站更快速。因此Socket.io使用了以下的javascript cdn :
```html
<script src="https://cdn.socket.io/socket.io-1.3.4.js"></script>
```
但是socket.io server 會在 polling的請求回覆400 Bad request

解決方法是從Socket.io依賴的原始碼找出return Access-Control-Allow-Origin Header的地方,根據socket.io v1.3.5 的源始碼,return Access-Control-Allow-Origin Header的地方位於node_modules/socket.io/node_modules/engine.io/lib/transports/polling-xhr.js

```javascript
/**
 * Returns headers for a response.
 *
 * @param {http.ServerRequest} request
 * @param {Object} extra headers
 * @api private
 */

XHR.prototype.headers = function(req, headers){
  headers = headers || {};

  if (req.headers.origin) {
    headers['Access-Control-Allow-Credentials'] = 'true';
    headers['Access-Control-Allow-Origin'] = req.headers.origin;
  } else {
    headers['Access-Control-Allow-Origin'] = '*';
  }

  this.emit('headers', headers);
  return headers;
};
```
將該function 改成
```javascript
/**
 * Returns headers for a response.
 *
 * @param {http.ServerRequest} request
 * @param {Object} extra headers
 * @api private
 */

XHR.prototype.headers = function(req, headers){
  headers = headers || {};

  if (req.headers.origin) {
    headers['Access-Control-Allow-Credentials'] = 'true';
    if(req.headers.origin == "https://www.example-domain.com/"){
        headers['Access-Control-Allow-Origin'] = "https://cdn.socket.io/";
    }else{
        headers['Access-Control-Allow-Origin'] = req.headers.origin;
    }
  } else {
    headers['Access-Control-Allow-Origin'] = '*';
  }

  this.emit('headers', headers);
  return headers;
};
```
或者直接新增　```headers['Access-Control-Allow-Origin'] = "https://cdn.socket.io/";```　雖然修改方法很醜。但至少可以令網站使用Socket.io CDN
