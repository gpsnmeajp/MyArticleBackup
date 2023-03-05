うっほあー、USB関連勉強した後だとめっちゃ簡単、というか、libusbそのまんまやんこれ
ベンダーIDも変更してみるか
Microsoft OS 2.0 Descriptorが自動認識の鍵になっているのはわかった。Win8.1以降から使えるらしい。

```html:webusb.htm
<html>
Hello<br>
<script src="webusb.js"></script>
<input type="button" onClick="connectbutton()" value="接続">
</html>
```

```js:webusb.js
function connectbutton()
{
	var device;
	var buf = new ArrayBuffer(1);
	var dat = new Uint8Array(buf, 0, buf.length);
	dat[0] = 0x81;
		
	navigator.usb.requestDevice({ filters: [{ vendorId: 0x04D8 , productId: 0x0053}] })
	.then(selectedDevice => {
	   device = selectedDevice;
	   return device.open(); // Begin a session.
	 })
//	.then(() => device.selectConfiguration(0)) // Select configuration #1 for the device.
	.then(() => device.claimInterface(0)) // Request exclusive control over interface #2.
	.then(() => device.transferOut(1, dat)) // Waiting for 64 bytes of data from endpoint #5.
	.then(() => device.transferIn(1, 64)) // Waiting for 64 bytes of data from endpoint #5.
	.then(result => {
		for(i=0;i<result.data.byteLength;i++)
		{
			console.log('Received: ' + result.data.getInt8(i));
		}
	})
	.catch(error => { console.log(error); });
}
```

#動作サンプル
もうちょっとマシなやつを実際に動く形で置いてみました．
Chromeの最新版をお使いの場合は、以下のページのWebUSBで確認ができます．
https://sabowl.sakura.ne.jp/gpsnmeajp/webusb/webusb.htm



#参考文献
Access USB Devices on the Web
https://developers.google.com/web/updates/2016/03/access-usb-devices-on-the-web
