chrome 默认情况下无法走系统代理, 需要编辑 `启动chrome应用参数` , `添加代理服务器`

```bash
sudo vim /usr/share/applications/google-chrome.desktop
# Exec=/usr/bin/google-chrome-stable %U
Exec=/usr/bin/google-chrome-stable --proxy-server="http://127.0.0.1:7897" %U
```