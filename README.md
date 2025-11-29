# Snake
Implement snake Game on FPGA

**Demo video**: https://youtu.be/7CC-fl-74z8?si=TBL-f0BjG2x86DW3

# 簡介
數位系統實驗期末project,利用FPGA上的按鈕搭配VGA輸出來玩貪食蛇

# 模組
- **clk_div_3.v:** 3.3HZ
- **clk_div_25M.v:** 25MHZ
- **Snake.v:** 控制蛇的移動，並且輸出當前的遊戲狀態
- **vga_main.v:** 控制VGA中V_sync訊號的輸出
- **vga_aux.v:** 控制VGA中H_sync訊號的輸出，並且將遊戲的狀態轉換成VGA訊號並輸出
- **SevenDisplay.v:** 顯示得分到七段顯示器上
- **random.v:** 生成兩組隨機數，一個用於蘋果的位置，另一個用於障礙物的位置
- **firework.v:** 利用dot matrix顯示圖案，而圖案會根據狀態(輸、贏和得分)而有所不同

## Snake.v
### 輸出
- **snake(72bit):** 一共包含1個頭跟8個身體的位置，一個位置用8個bit表示，共計9 * 8 = 72bit
- **barrier , apple(8bit):** 障礙物與蘋果的位置，各8bit
- **score(4bit):** 當前得分
- **dead_flag, score_flag, win_flag:** 當前遊戲狀態(得分、輸和贏)
### snake輸出格式
每8個bit能儲存一個座標(0~100)，目前蛇身暫定最長為8，可視情況增加
![snake](https://github.com/Potassium-chromate/Snake/blob/main/picture/Snake.png)

### 網格介紹
一共是8*8的網格(黃色的部分)，其中白色設為出界區。
![grid](https://github.com/Potassium-chromate/Snake/blob/main/picture/grid.png)

### 狀態機介紹
* **reset:**
  - 將得分設置為0
  - 蛇頭位置設置在12(網格左上角)
  - 所有蛇身位置設為0
* **head_renew:**
  - 決定蛇頭下一步要往哪邊移動
* **check:**
  - 檢查蛇頭是否出界
  - 檢查蛇頭是否吃到蘋果
* **move:**
  - 移動蛇頭
  - 移動蛇身
* **check_body:**
  - 檢查蛇頭是否咬到蛇身
![FSM](https://github.com/Potassium-chromate/Snake/blob/main/picture/FSM.png)

## vga_main.v　
### 輸入
- **clk:** 當`vga_aux.v` 每完成一列(800 pixel)會輸出一個clock，而`vga_main.v`會在該clock內完成換列的輸出
### 輸出
- **V_sync:** 
- **row_count(10bit):** 用於計算目前輸出到第幾列(一張畫面共480列)

## vga_aux.v　
### 輸入
- **snake(72bit):** 來自`snake.v`的輸出
- **row_count(10bit):** 來自`vga_main.v`的輸出
- **apple, barrier(8bit):** 來自`snake.v`的輸出
### 輸出
- **H_sync, work_clk**
- **red_out, green_out, blue_out**
### 過程
1. 先將`snake`、`apple`、`barrier`映射到8*8個的網格上。完成後，如果該格上有東西則該格為1'b1，否則為1'b0
2. 利用`row_count`和`col_count`來得知目前輸出的RGB值會對應到螢幕上的哪個位置
3. 判斷應該要輸出8*8網格上的哪一個格子
4. 根據網格上的物件判斷輸出顏色: 
    * **無:** 黑色
    * **遊戲邊界:** 黃色
    * **蛇頭:** 綠色
    * **蛇身:** 白色
    * **蘋果:** 紅色
    * **障礙物:** 藍色
## firework.v
### 輸入
- **score_flag, dead_flag, win_flag:** 來自`snake.v`的輸出
### 輸出
- **col_1, col_2, row(8bit)**
### 輸出樣式
- **吃到蘋果:** 笑臉 & 愛心
- **碰到障礙物:** 叉叉 & 怒臉
- **到達分數上限(9分):** win
