# Final-project
這是一個Quartus去寫Verilog的閃避遊戲
分為new版和原版，new版更改以下功能:角色為兩點綠點，子彈數量增加至6，end game畫面更改為X

請多利用注釋，注釋內容已足夠完整和清晰
我們的閃避遊戲功能包括(在影片中有講解):
子彈:子彈的作用為把該行的方塊推上去，會有燈顯示數量
結束遊戲:GG為結束遊戲文字
難度控制器:以指撥開關為控制
控制:s1和s3分別為控制人物的左右移動，s2為發射子彈，s4為重新開始遊戲
角色:角色為8X8顯示器中最下面綠色的光點
障礙物:為8X8中會用偽隨機分配的速度以紅點為顯示
分數:會隨時間的變動而增加，在困難難度開啟後，分數會上升6倍(困難難度速度提升兩倍，分數隨時間上升3倍，𠔏6倍)
