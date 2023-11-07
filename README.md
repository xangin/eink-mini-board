# eink-mini-board

<a href="https://www.buymeacoffee.com/xangin" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Coffee" height="30" width="130"></a>

本專案為利用2.9"電子紙搭配ESPhome與Home Assistant，平常顯示為時鐘與溫溼度

透過HA控制換頁，加上任何想要顯示的資訊，例如當衣服洗好時，會自動將畫面切為訊息提示

要有多少頁或是顯示什麼都能自由設計


- 時鐘頁面 (溫溼度為讀取自外掛SHT31)

<img src="https://user-images.githubusercontent.com/56766371/280905433-d5fae821-7332-4d35-b14f-b55f42dde0f8.jpg" width="66%" />

- 訊息頁面

<img src="https://user-images.githubusercontent.com/56766371/280905123-8fde26c9-1ff2-4ab5-92ce-a17f62dd761a.jpg" width="66%" />


## Installation 安裝方式

1. 將`/fonts`資料夾內的檔案及`eink-mini-board.yaml`放到HA/config/esphome的資料夾內
2. 將`eink-mini-board.yaml`的內容修改成自己想要的內容 **解說在下方**
3. 在ESPhome編譯成功後，將`eink-mini-board.yaml`燒錄或OTA至模組
4. 完成!


## ESPHome yaml 說明

### 在HA控制換頁

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

這邊有2個按鈕，按下去分別會去顯示p1與p2，如果有要再新增更多頁可以再仿照程式碼再新增


