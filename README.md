# eink-mini-board

<a href="https://www.buymeacoffee.com/xangin" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Coffee" height="30" width="130"></a>

本專案為利用2.9"電子紙搭配ESPhome與Home Assistant，平常顯示為時鐘與溫溼度

透過HA控制換頁，加上任何想要顯示的資訊，例如當衣服洗好時，會自動將畫面切為訊息提示

要有多少頁或是顯示什麼都能自由設計

- 成品尺寸: 寬 88mm x 高 47mm

<img src="https://github.com/xangin/eink-mini-board/assets/56766371/fb95f3e1-b45d-43b8-a1e2-9ec430b8b43e" width="50%" />

- 成品背面

<img src="https://github.com/xangin/eink-mini-board/assets/56766371/12abbdc1-18ad-423c-b0ec-0855c366c5ef" width="50%" />

- 時鐘頁面 (溫溼度為讀取自外掛SHT31)

<img src="https://user-images.githubusercontent.com/56766371/280905433-d5fae821-7332-4d35-b14f-b55f42dde0f8.jpg" width="50%" />

- 訊息頁面

<img src="https://user-images.githubusercontent.com/56766371/280905123-8fde26c9-1ff2-4ab5-92ce-a17f62dd761a.jpg" width="50%" />

## Hardware 硬體架構

- [微雪 2.9吋黑白墨水屏裸屏](https://detail.tmall.com/item.htm?id=605757420567) - 不帶外殼
- [电子墨水屏外壳(EW029F2(2.9寸单电池))](https://item.taobao.com/item.htm?id=601700008521) - 外殼
- [溫溼度模組](https://item.taobao.com/item.htm?id=581637366281) - SHT30

**電路板**

- [墨水屏通用轉接板](https://oshwhub.com/lingdy2012/mo-shui-ping-tong-yong-zhuan-jie-ban-_0603_wos_v0-1) - 閑魚有售
- [ESP32-C3 開發板 ESP32 SuperMini](https://item.taobao.com/item.htm?id=707413078834)
- 自製電路連接上面兩片電路

<img src="https://github.com/xangin/eink-mini-board/assets/56766371/99e41766-5d21-4ef5-8a39-22a8d514a38d"  width="50%" />

| I/O定義 | Pin name  |	ESP32C3 |
|:----:|:----:|:----:|
| SHT30 |	SDA |	GPIO4 |
|	|SCL	| GPIO5 |
| | 3V | 3V |
| eink |	CS |	GPIO7|
| |	DC |	GPIO6 |
| |	CLK |	GPIO2 |
| |	MOSI |	GPIO3 |
| | 3V | 3V |

## Installation 安裝方式

1. 將`/fonts`資料夾內的檔案及`eink-mini-board.yaml`放到HA/config/esphome的資料夾內
2. 將`eink-mini-board.yaml`的內容修改成自己想要的內容 **解說在下方**
3. 在ESPhome編譯成功後，將`eink-mini-board.yaml`燒錄或OTA至模組
4. 完成!


## ESPHome yaml 說明

### 在HA控制換頁

有2個按鈕，按下去分別會去顯示p1(Time Page)與p2(Message Page)，如果有要再新增更多頁可以再仿照程式碼再新增

```YAML
button:
  - platform: template
    name: "Show Time Page"
    icon: 'mdi:clock'
    on_press:
      then:
        - display.page.show: p1
        - component.update: my_display
    
  - platform: template
    name: "Show Message Page"
    icon: 'mdi:update'
    on_press:
      then:
        - display.page.show: p2
        - component.update: my_display
```

### 根據Wi-Fi強度顯示圖示

說明: 
- 大於等於-60顯示三格
- -60~-70顯示兩格
- -70~-75顯示一格
- -75~-85顯示零格
- 小於-85顯示中斷

可自由變更強度範圍要顯示的格數

```YAML
          //wifi signal
          if (id(wifisignal).state >= -60) {
              //Excellent
              it.print(0, 0, id(wifi_font), "\U000F08BE");
          } else if (id(wifisignal).state  >= -70) {
              //Good
              it.print(0, 0, id(wifi_font), "\U000F08BD");
          } else if (id(wifisignal).state  >= -75) {
              //Fair
              it.print(0, 0, id(wifi_font),"\U000F08BC");
          } else if (id(wifisignal).state  >= -85) {
              //Weak
              it.print(0, 0, id(wifi_font),"\U000F08BF");
          } else {
              //Unlikely working signal
              it.print(0, 0, id(wifi_font),"\U000F0783");
          }
```

### 修改自訂訊息內容

作法:

1. 在字型宣告處的`msg_font`將要顯示的中文字**先全部寫出來**這樣才能正常顯示唷!!
```YAML
font:
  - file: "fonts/NotoSansTC-Medium.ttf"
    id: msg_font
    size: 40
    glyphs: 衣服已經洗拿去烘好囉!趕快收起來ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz,."%-~_:°
```
2. 在`display`的p2，置換想要顯示的文字，依目前設定的大小，一行就是顯示`7個中文字`，超過將無法顯示唷!
```YAML
display:
  ...
      - id: p2
        lambda: |- 
          it.printf(150,15, id(msg_font), TextAlign::TOP_CENTER, "衣服已經洗好囉!");
          it.printf(150,70, id(msg_font), TextAlign::TOP_CENTER, "趕快拿去烘~");
```

### 增加更多頁面

作法: 仿照按鈕及顯示的程式碼，可再新增多組，例如下面是第三頁，按下去顯示衣服烘好了

有不在`msg_font`內的中文字要記得新增，這樣才能正常顯示唷!

```YAML

button: #複製在button程式碼的最下面，不可重複寫button唷!
  ...
  - platform: template
    name: "Show Dryer Done Page"
    icon: 'mdi:update'
    on_press:
      then:
        - display.page.show: p3
        - component.update: my_display


display: #複製在display程式碼的最下面，不可重複寫display唷!
  ...
      - id: p3
        lambda: |- 
          it.printf(150,15, id(msg_font), TextAlign::TOP_CENTER, "衣服已經烘好囉!");
          it.printf(150,70, id(msg_font), TextAlign::TOP_CENTER, "趕快收起來!!");
```
