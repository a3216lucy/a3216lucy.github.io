---
title: WebSocket 介紹與使用 
date: 2023-08-22 21:07:27
categories: 
- 技術筆記
tags:
- webpack
---
在介紹 WebSocket 之前，先來了解幾個 HTTP 的連線方式。

## HTTP 的連線方式

### Polling

這種方式是早期透過 Javascript 定時（`setInterval`、`setTimeout`）來向後端請求最新資料並呈現於頁面之上， 這麼做的好處是容易實現， 但我們並不知曉 Server 端何時會更新， 只能**傻傻的定時獲取**， 造成的影響就是<U>頻寬的浪費及資料呈現不即時</U>。

![](https://miro.medium.com/v2/resize:fit:1382/1*HJww54OsxD5D3Cn5YyJunQ.png)

### Long-Polling（長時間輪詢）

這種方式就是收到前端請求後，Server 端會等待， 若這段時間有資料就會將最新資料回傳給前端， 因此等待的這段期間 Client 什麼事情也不用做，等待資料回傳後再發送下一個請求， 雖然長時間的連接解決了 Polling 開銷的問題，<u>但如果在更新資料很頻繁的狀況下，也會造成連續的 Polling 的動作產生，**效能未必較佳**</u>。

![](https://miro.medium.com/v2/resize:fit:1204/1*kseayTfoDyTzI8uFSen9Cg.png)

### Server Sent Events

簡單來說就是 <u>**伺服端單向的推送資料給瀏覽器**</u>，實作起來相對容易。
但這種方式較少被使用，  因為 SSE 能做的功能 Websocket 也能完成， 差別只是雙向溝通還是單向溝通而已。
不過在某些應用上，瀏覽器不太需要主動發送資料給 Server 端（例：股市行情、即時新聞…），只需要接收 Server 端的最新資料即可，就適合使用。

![](https://miro.medium.com/v2/resize:fit:1400/1*9akWJheWpyJuTvo6AfoEow.png)

### Websocket

以上幾個方式的共同點就是都基於 HTTP 進行傳輸，而我們也知道 HTTP 的傳輸為了確保正確性會帶有較多的標頭…等資訊；而 **Websocket 只有一開始確定連線時會走 HTTP 以外，之後的傳輸都不是基於 HTTP**，在傳輸上會來的更有效率，且提供了**雙向溝通**的功能，避免延遲，因此需要即時互動的應用就非常適合使用 Websocket 的方式來進行實作。

![](https://miro.medium.com/v2/resize:fit:1106/1*xPnOxyaKv5NleFy_13kCqg.png)

## WebSocket 和 HTTP 的差異

和 `HTTP` 最大的差別是，<u>`WebScoket` 在建立連線之後，可以實現 client 與 server 的雙向溝通機制</u>，也就是說，雙方可以即時的交換資訊，<u>不用再等待 client 發出請求，或使用佔用資源的輪詢方式來實現</u>。

另一方面，在建立連線之後就會持續保持著連線狀態，所以每次通訊時所發出的 message，就可以省略掉 headers 當中一些關於連線狀態的資訊，因而效率高了不少。

### WebSocket 與 HTTP ：

- 同屬於應用層， 都是基於 TCP 來進行連線傳輸
- 默認的 port 也是為 80 和 443，並透過同樣的 HTTP 3-way handshake 建立連線
- 與 HTTP 一樣可以進行加密傳輸， HTTP 的部份為 `https`， WebSocket 則為 `wss`。
- 但 WebSocket 真正傳輸資料時就不必經過 HTTP 的方式（要進行像是 HTTPS 當中的 SSL/TLS 的加密安全傳輸，有 `WebSocket Secure` 協議可以使用）

![](https://miro.medium.com/v2/resize:fit:696/format:webp/1*Aefa0WHo_VsgQMjvj9rPFg.png)

## WebSocket 的使用

WebSocket 在使用上其實不難。[MDN](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket) 上有關於 WebSocket API (client 端) 的範例。

這裡我們來簡單建立一下連線：

```javascript
// Create WebSocket connection.
const socket = new WebSocket("wss://echo.websocket.org");

// Connection opened
socket.addEventListener('open', function (event) {
    console.log('Open connection')
    socket.send('Hello Server!');
});

// Listen for messages
socket.addEventListener('message', function (event) {
    console.log(`Message from server: ${event.data}`);
});
```

如果把上面這段程式碼貼到 DevTool 的 console 當中，就可以看到其連線運作。

內容其實相當簡單，首先建立 WebSocket 連線實例，接著，就可以監聽各種 events，像是

* open （連線開啟）
* close （連線關閉）
* error （連線錯誤）
* message （連線訊息）

之後，再根據不同的 events 做出不同的對應方式，譬如在畫面上呈現連線訊息等。

除了監聽事件之外，也可以透過 `socket.send()` 來送出資訊，以及 `send.close()` 來關閉連線。

MDN 上介紹的 WebSocket API 其實就可以幫助我們快速建立 `WebSocket` 連線，不過如果我們想要在自己的專案當中進行更複雜的操作，也可以找到很完整的套件來幫助我們（例如： [socket.io](https://socket.io/)），可以同時用在 client 端與 server 端。

---

有了 `WebSocket` 讓我們的應用程式可以和使用者有更多的互動，除了「聊天室」的應用之外，還有其他的應用場景像是「訊息推播」、「即時訊息更新」、「共同編輯」等等，應用相當廣泛。

## Ref

* [WebSocket](https://en.wikipedia.org/wiki/WebSocket#History)
* [WebSocket (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)
