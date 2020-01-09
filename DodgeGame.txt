//======================================================= KeyDebounce Module ===================================
module KEY_Debounce( CLK, KEY_In, KEY_Out );//KEY_Out is posedge
	input KEY_In,CLK;
	output KEY_Out;
   reg [2:0]  buttonReg;
	 always @(posedge CLK)
		buttonReg <= {buttonReg[1:0], KEY_In};
	  assign KEY_Out = buttonReg[2:1] == 2'b01;
		  //neg_edge = button_reg[2:1] == 2'b10;
endmodule
//======================================================= SpeedModule =========================================
module SpeedRam(CLK,Speed,SpeedState,HARDMODE);
	input CLK,HARDMODE;
	output reg Speed;
	input [1:0]SpeedState;
	reg[12:0] speedCount;
	reg[11:0] hardSpeedCount;
	initial begin
		speedCount = 13'b0000000000000;
		hardSpeedCount = 12'b000000000000;
		Speed = 1'b0;
	end
	always@(posedge CLK)begin
	speedCount <= speedCount + 1'b1;
	hardSpeedCount <= hardSpeedCount + 1'b1;
		if(HARDMODE == 1'b0)begin
			if(speedCount == 13'b0000000000000&&(SpeedState == 2'b00||SpeedState == 2'b01||SpeedState==2'b10||SpeedState == 2'b11))begin
				Speed <= 1'b1;//While Speed is 1 = Trigger Time Point
			end
			else if (speedCount == 13'b1000000000000&&(SpeedState == 2'b01||SpeedState==2'b10||SpeedState == 2'b11))begin
				Speed <= 1'b1;
			end
			else if (speedCount == 13'b0100000000000&&SpeedState == 2'b11)begin
				Speed <= 1'b1;
			end
			else if (speedCount == 13'b1100000000000&&SpeedState == 2'b11)begin
				Speed <= 1'b1;
			end
			else begin
				Speed <= 1'b0;//Naturally Speed = 0
			end
		end
		else begin
			if(hardSpeedCount == 12'b000000000000&&(SpeedState == 2'b00||SpeedState == 2'b01||SpeedState==2'b10||SpeedState == 2'b11))begin
				Speed <= 1'b1;//While Speed is 1 = Trigger Time Point
			end
			else if (hardSpeedCount == 12'b100000000000&&(SpeedState == 2'b01||SpeedState==2'b10||SpeedState == 2'b11))begin
				Speed <= 1'b1;
			end
			else if (hardSpeedCount == 12'b010000000000&&SpeedState == 2'b11)begin
				Speed <= 1'b1;
			end
			else if (hardSpeedCount == 12'b110000000000&&SpeedState == 2'b11)begin
				Speed <= 1'b1;
			end
			else begin
				Speed <= 1'b0;//Naturally Speed = 0
			end
		end
	end
endmodule
//==================================================== Score Count ===================================================
module ScoreCount(CLK,SCORE,HARDMODE);//SCORE = 1 Output to count
	input CLK,HARDMODE;
	output reg SCORE;
	reg[8:0] speedCount;
	reg[7:0] hardSpeedCount;
	initial begin
		speedCount = 9'b000000000;
		hardSpeedCount = 8'b00000000;
		SCORE = 1'b0;
	end
	always@(posedge CLK)begin
	speedCount <= speedCount + 1'b1;
	hardSpeedCount <= hardSpeedCount + 1'b1;
		if(HARDMODE == 1'b0)begin
			if(speedCount == 9'b000000000)begin
				SCORE <= 1'b1;//While SCORE is 1 = Trigger Time Point
			end
			else begin
				SCORE <= 1'b0;//Naturally SCORE = 0
			end
		end
		else begin
			if(hardSpeedCount == 8'b00000000)begin
				SCORE <= 1'b1;//While SCORE is 1 = Trigger Time Point
			end
			else begin
				SCORE <= 1'b0;//Naturally SCORE = 0
			end
		end
	end
endmodule
//=============================================== Top Module ===========================================================
module DodgeGame(output reg [0:7] DATA_R, DATA_G, DATA_B,
					  output reg [3:0] COMM,
					  output reg [6:0] SegDisplay,
					  output reg [3:0] SegCOM,
					  input CLK,LEFT,RIGHT,MIDDLE,RESET,HARDMODE,  
					  output  pos_edge, neg_edge,Speed1,Speed2,Speed3,Speed4,Speed5,Speed6,Speed7,Speed8,SCORE,
					  output reg GameOver,
					  output reg [4:0] ammo
					  );

	//Top Module Initialization
	wire LEFTEDGE,MIDDLEEDGE,RIGHTEDGE,RESETEDGE;
	parameter LEFTLIMIT = 3'b000, RIGHTLIMIT = 3'b111;
	reg [2:0]Position,ObsVecPosition1,ObsVecPosition2,ObsVecPosition3,ObsVecPosition4,
				ObsVecPosition5,ObsVecPosition6,ObsVecPosition7,ObsVecPosition8;
	reg      Hit1,Hit2,Hit3,Hit4,Hit5,Hit6,Hit7,Hit8;
	reg[1:0] SpeedState1,SpeedState2,SpeedState3,SpeedState4,SpeedState5,SpeedState6,SpeedState7,SpeedState8;
	reg[3:0] Count;
	reg[3:0] ScoreDigit1,ScoreDigit2,ScoreDigit3,ScoreDigit4;
	
	initial
		begin
		DATA_R <= 8'b11111111; //0 is top, 7 is bottom
		DATA_G <= 8'b11111110;
		DATA_B <= 8'b11111111;
		
		ammo <= 5'b 11111;
		Hit1 <= 1'b0;Hit2<= 1'b0;Hit3<= 1'b0;Hit4<= 1'b0;Hit5<= 1'b0;Hit6<= 1'b0;Hit7<= 1'b0;Hit8<= 1'b0;
		GameOver <= 1'b0;
		Position <= 3'b100;
		
		ScoreDigit1 <= 4'b0000; ScoreDigit2 <= 4'b0000;
		ScoreDigit3 <= 4'b0000; ScoreDigit4 <= 4'b0000;
		
		SegDisplay <= 7'b0000001;
		SegCOM <= 4'b1110; //0 is current 7segment Displayer || Rightmost is LowerDigit
		
		ObsVecPosition1 <= 3'b000; ObsVecPosition2 <= 3'b000; ObsVecPosition3 <= 3'b000; ObsVecPosition4 <= 3'b000;
		ObsVecPosition5 <= 3'b000; ObsVecPosition6 <= 3'b000; ObsVecPosition7 <= 3'b000; ObsVecPosition8 <= 3'b000;
		
		SpeedState1 <= 2'b00;SpeedState2 <= 2'b01;SpeedState3 <= 2'b00;SpeedState4 <= 2'b10;
		SpeedState5 <= 2'b10;SpeedState6 <= 2'b01;SpeedState7 <= 2'b11;SpeedState8 <= 2'b11;
		COMM <= 4'b1011;
		end
	//27 = EN S2 S1 S0 = Code of colum instance
	//000 col 1
	//001 col 2
	//010 col 3
	//011 col 4
	//100 col 5
	//101 col 6
	//110 col 7
	//111 col 8
	//=================================================== Displaying LEDS ==============================================================
	always @(posedge CLK)
		begin
		if(RESETEDGE == 1'b0)begin
			if(GameOver == 1'b1)begin
				if(Count == 4'b0001)begin
					COMM <= 4'b1000;DATA_R <= 8'b11111111;DATA_G <= 8'b10000001;DATA_B <= 8'b11111111;
					case({ScoreDigit1} ) //AfterGame Score Display Digit 1
					4'b0000:begin SegCOM <= 4'b1110;SegDisplay <= 7'b0000001; end
					4'b0001:begin SegCOM <= 4'b1110;SegDisplay <= 7'b1001111; end
					4'b0010:begin SegCOM <= 4'b1110;SegDisplay <= 7'b0010010; end
					4'b0011:begin SegCOM <= 4'b1110;SegDisplay <= 7'b0000110; end
					4'b0100:begin SegCOM <= 4'b1110;SegDisplay <= 7'b1001100; end
					4'b0101:begin SegCOM <= 4'b1110;SegDisplay <= 7'b0100100; end
					4'b0110:begin SegCOM <= 4'b1110;SegDisplay <= 7'b0100000; end
					4'b0111:begin SegCOM <= 4'b1110;SegDisplay <= 7'b0001111; end
					4'b1000:begin SegCOM <= 4'b1110;SegDisplay <= 7'b0000000; end
					4'b1001:begin SegCOM <= 4'b1110;SegDisplay <= 7'b0000100; end
					default:begin SegCOM <= 4'b1110;SegDisplay <= 7'b0000001; end
					endcase
				end
				else if(Count == 4'b0010)begin
					COMM <= 4'b1001;DATA_R <= 8'b11111111;DATA_G <= 8'b10111101;DATA_B <= 8'b11111111;
					case({ScoreDigit2} ) //AfterGame Score Display Digit 2
					4'b0000:begin SegCOM <= 4'b1101;SegDisplay <= 7'b0000001; end
					4'b0001:begin SegCOM <= 4'b1101;SegDisplay <= 7'b1001111; end
					4'b0010:begin SegCOM <= 4'b1101;SegDisplay <= 7'b0010010; end
					4'b0011:begin SegCOM <= 4'b1101;SegDisplay <= 7'b0000110; end
					4'b0100:begin SegCOM <= 4'b1101;SegDisplay <= 7'b1001100; end
					4'b0101:begin SegCOM <= 4'b1101;SegDisplay <= 7'b0100100; end
					4'b0110:begin SegCOM <= 4'b1101;SegDisplay <= 7'b0100000; end
					4'b0111:begin SegCOM <= 4'b1101;SegDisplay <= 7'b0001111; end
					4'b1000:begin SegCOM <= 4'b1101;SegDisplay <= 7'b0000000; end
					4'b1001:begin SegCOM <= 4'b1101;SegDisplay <= 7'b0000100; end
					default:begin SegCOM <= 4'b1101;SegDisplay <= 7'b0000001; end
					endcase
				end
				else if(Count == 4'b0011)begin
					COMM <= 4'b1010;DATA_R <= 8'b11111111;DATA_G <= 8'b10100101;DATA_B <= 8'b11111111;
					case({ScoreDigit3} ) //AfterGame Score Display Digit 3
					4'b0000:begin SegCOM <= 4'b1011;SegDisplay <= 7'b0000001; end
					4'b0001:begin SegCOM <= 4'b1011;SegDisplay <= 7'b1001111; end
					4'b0010:begin SegCOM <= 4'b1011;SegDisplay <= 7'b0010010; end
					4'b0011:begin SegCOM <= 4'b1011;SegDisplay <= 7'b0000110; end
					4'b0100:begin SegCOM <= 4'b1011;SegDisplay <= 7'b1001100; end
					4'b0101:begin SegCOM <= 4'b1011;SegDisplay <= 7'b0100100; end
					4'b0110:begin SegCOM <= 4'b1011;SegDisplay <= 7'b0100000; end
					4'b0111:begin SegCOM <= 4'b1011;SegDisplay <= 7'b0001111; end
					4'b1000:begin SegCOM <= 4'b1011;SegDisplay <= 7'b0000000; end
					4'b1001:begin SegCOM <= 4'b1011;SegDisplay <= 7'b0000100; end
					default:begin SegCOM <= 4'b1011;SegDisplay <= 7'b0000001; end
					endcase
				end
				else if(Count == 4'b0100)begin
					COMM <= 4'b1011;DATA_R <= 8'b11111111;DATA_G <= 8'b10100001;DATA_B <= 8'b11111111;
					case({ScoreDigit4} ) //AfterGame Score Display Digit 4
					4'b0000:begin SegCOM <= 4'b0111;SegDisplay <= 7'b0000001; end
					4'b0001:begin SegCOM <= 4'b0111;SegDisplay <= 7'b1001111; end
					4'b0010:begin SegCOM <= 4'b0111;SegDisplay <= 7'b0010010; end
					4'b0011:begin SegCOM <= 4'b0111;SegDisplay <= 7'b0000110; end
					4'b0100:begin SegCOM <= 4'b0111;SegDisplay <= 7'b1001100; end
					4'b0101:begin SegCOM <= 4'b0111;SegDisplay <= 7'b0100100; end
					4'b0110:begin SegCOM <= 4'b0111;SegDisplay <= 7'b0100000; end
					4'b0111:begin SegCOM <= 4'b0111;SegDisplay <= 7'b0001111; end
					4'b1000:begin SegCOM <= 4'b0111;SegDisplay <= 7'b0000000; end
					4'b1001:begin SegCOM <= 4'b0111;SegDisplay <= 7'b0000100; end
					default:begin SegCOM <= 4'b0111;SegDisplay <= 7'b0000001; end
					endcase
				end
				else if(Count == 4'b0101)begin
					COMM <= 4'b1100;DATA_R <= 8'b11111111;DATA_G <= 8'b11111111;DATA_B <= 8'b10000001;
				end
				else if(Count == 4'b0110)begin
					COMM <= 4'b1101;DATA_R <= 8'b11111111;DATA_G <= 8'b11111111;DATA_B <= 8'b10111101;
				end
				else if(Count == 4'b0111)begin
					COMM <= 4'b1110;DATA_R <= 8'b11111111;DATA_G <= 8'b11111111;DATA_B <= 8'b10100101;
				end
				else if(Count == 4'b1000)begin
					COMM <= 4'b1111;DATA_R <= 8'b11111111;DATA_G <= 8'b11111111;DATA_B <= 8'b10100001;
				end
			end
			else if(Count == 4'b0001)begin
				case({Position} ) //Player Position && ScoreDigit1
					3'b000:begin COMM <= 4'b1000;DATA_G <= 8'b11111110;DATA_R <= 8'b11111111; end
					3'b001:begin COMM <= 4'b1001;DATA_G <= 8'b11111110;DATA_R <= 8'b11111111; end
					3'b010:begin COMM <= 4'b1010;DATA_G <= 8'b11111110;DATA_R <= 8'b11111111; end
					3'b011:begin COMM <= 4'b1011;DATA_G <= 8'b11111110;DATA_R <= 8'b11111111; end
					3'b100:begin COMM <= 4'b1100;DATA_G <= 8'b11111110;DATA_R <= 8'b11111111; end
					3'b101:begin COMM <= 4'b1101;DATA_G <= 8'b11111110;DATA_R <= 8'b11111111; end
					3'b110:begin COMM <= 4'b1110;DATA_G <= 8'b11111110;DATA_R <= 8'b11111111; end
					3'b111:begin COMM <= 4'b1111;DATA_G <= 8'b11111110;DATA_R <= 8'b11111111; end
					default:COMM = COMM;
				endcase
				case({ScoreDigit1} )
					4'b0000:begin SegCOM <= 4'b1110;SegDisplay <= 7'b0000001; end
					4'b0001:begin SegCOM <= 4'b1110;SegDisplay <= 7'b1001111; end
					4'b0010:begin SegCOM <= 4'b1110;SegDisplay <= 7'b0010010; end
					4'b0011:begin SegCOM <= 4'b1110;SegDisplay <= 7'b0000110; end
					4'b0100:begin SegCOM <= 4'b1110;SegDisplay <= 7'b1001100; end
					4'b0101:begin SegCOM <= 4'b1110;SegDisplay <= 7'b0100100; end
					4'b0110:begin SegCOM <= 4'b1110;SegDisplay <= 7'b0100000; end
					4'b0111:begin SegCOM <= 4'b1110;SegDisplay <= 7'b0001111; end
					4'b1000:begin SegCOM <= 4'b1110;SegDisplay <= 7'b0000000; end
					4'b1001:begin SegCOM <= 4'b1110;SegDisplay <= 7'b0000100; end
					default:SegCOM = SegCOM;
				endcase
			end
			else if(Count == 4'b0010)begin
				case({ObsVecPosition1} ) //RedDot 1 position && ScoreDigit2
					3'b000:begin COMM <= 4'b1000;DATA_R <= 8'b01111111;DATA_G <= 8'b11111111; end
					3'b001:begin COMM <= 4'b1000;DATA_R <= 8'b10111111;DATA_G <= 8'b11111111; end
					3'b010:begin COMM <= 4'b1000;DATA_R <= 8'b11011111;DATA_G <= 8'b11111111; end
					3'b011:begin COMM <= 4'b1000;DATA_R <= 8'b11101111;DATA_G <= 8'b11111111; end
					3'b100:begin COMM <= 4'b1000;DATA_R <= 8'b11110111;DATA_G <= 8'b11111111; end
					3'b101:begin COMM <= 4'b1000;DATA_R <= 8'b11111011;DATA_G <= 8'b11111111; end
					3'b110:begin COMM <= 4'b1000;DATA_R <= 8'b11111101;DATA_G <= 8'b11111111; end
					3'b111:begin COMM <= 4'b1000;DATA_R <= 8'b11111110;DATA_G <= 8'b11111111; end
					default:COMM = COMM;
				endcase
				case({ScoreDigit2} )
					4'b0000:begin SegCOM <= 4'b1101;SegDisplay <= 7'b0000001; end
					4'b0001:begin SegCOM <= 4'b1101;SegDisplay <= 7'b1001111; end
					4'b0010:begin SegCOM <= 4'b1101;SegDisplay <= 7'b0010010; end
					4'b0011:begin SegCOM <= 4'b1101;SegDisplay <= 7'b0000110; end
					4'b0100:begin SegCOM <= 4'b1101;SegDisplay <= 7'b1001100; end
					4'b0101:begin SegCOM <= 4'b1101;SegDisplay <= 7'b0100100; end
					4'b0110:begin SegCOM <= 4'b1101;SegDisplay <= 7'b0100000; end
					4'b0111:begin SegCOM <= 4'b1101;SegDisplay <= 7'b0001111; end
					4'b1000:begin SegCOM <= 4'b1101;SegDisplay <= 7'b0000000; end
					4'b1001:begin SegCOM <= 4'b1101;SegDisplay <= 7'b0000100; end
					default:SegCOM = SegCOM;
				endcase
			end
			else if(Count == 4'b0011)begin
				case({ObsVecPosition2} ) //RedDot 2 position && ScoreDigit3
					3'b000:begin COMM <= 4'b1001;DATA_R <= 8'b01111111;DATA_G <= 8'b11111111; end
					3'b001:begin COMM <= 4'b1001;DATA_R <= 8'b10111111;DATA_G <= 8'b11111111; end
					3'b010:begin COMM <= 4'b1001;DATA_R <= 8'b11011111;DATA_G <= 8'b11111111; end
					3'b011:begin COMM <= 4'b1001;DATA_R <= 8'b11101111;DATA_G <= 8'b11111111; end
					3'b100:begin COMM <= 4'b1001;DATA_R <= 8'b11110111;DATA_G <= 8'b11111111; end
					3'b101:begin COMM <= 4'b1001;DATA_R <= 8'b11111011;DATA_G <= 8'b11111111; end
					3'b110:begin COMM <= 4'b1001;DATA_R <= 8'b11111101;DATA_G <= 8'b11111111; end
					3'b111:begin COMM <= 4'b1001;DATA_R <= 8'b11111110;DATA_G <= 8'b11111111; end
					default:COMM = COMM;
				endcase
				case({ScoreDigit3} )
					4'b0000:begin SegCOM <= 4'b1011;SegDisplay <= 7'b0000001; end
					4'b0001:begin SegCOM <= 4'b1011;SegDisplay <= 7'b1001111; end
					4'b0010:begin SegCOM <= 4'b1011;SegDisplay <= 7'b0010010; end
					4'b0011:begin SegCOM <= 4'b1011;SegDisplay <= 7'b0000110; end
					4'b0100:begin SegCOM <= 4'b1011;SegDisplay <= 7'b1001100; end
					4'b0101:begin SegCOM <= 4'b1011;SegDisplay <= 7'b0100100; end
					4'b0110:begin SegCOM <= 4'b1011;SegDisplay <= 7'b0100000; end
					4'b0111:begin SegCOM <= 4'b1011;SegDisplay <= 7'b0001111; end
					4'b1000:begin SegCOM <= 4'b1011;SegDisplay <= 7'b0000000; end
					4'b1001:begin SegCOM <= 4'b1011;SegDisplay <= 7'b0000100; end
					default:SegCOM = SegCOM;
				endcase
			end
			else if(Count == 4'b0100)begin
				case({ObsVecPosition3} ) //RedDot 3 position && ScoreDigit4
					3'b000:begin COMM <= 4'b1010;DATA_R <= 8'b01111111;DATA_G <= 8'b11111111; end
					3'b001:begin COMM <= 4'b1010;DATA_R <= 8'b10111111;DATA_G <= 8'b11111111; end
					3'b010:begin COMM <= 4'b1010;DATA_R <= 8'b11011111;DATA_G <= 8'b11111111; end
					3'b011:begin COMM <= 4'b1010;DATA_R <= 8'b11101111;DATA_G <= 8'b11111111; end
					3'b100:begin COMM <= 4'b1010;DATA_R <= 8'b11110111;DATA_G <= 8'b11111111; end
					3'b101:begin COMM <= 4'b1010;DATA_R <= 8'b11111011;DATA_G <= 8'b11111111; end
					3'b110:begin COMM <= 4'b1010;DATA_R <= 8'b11111101;DATA_G <= 8'b11111111; end
					3'b111:begin COMM <= 4'b1010;DATA_R <= 8'b11111110;DATA_G <= 8'b11111111; end
					default:COMM = COMM;
				endcase
				case({ScoreDigit4} )
					4'b0000:begin SegCOM <= 4'b0111;SegDisplay <= 7'b0000001; end
					4'b0001:begin SegCOM <= 4'b0111;SegDisplay <= 7'b1001111; end
					4'b0010:begin SegCOM <= 4'b0111;SegDisplay <= 7'b0010010; end
					4'b0011:begin SegCOM <= 4'b0111;SegDisplay <= 7'b0000110; end
					4'b0100:begin SegCOM <= 4'b0111;SegDisplay <= 7'b1001100; end
					4'b0101:begin SegCOM <= 4'b0111;SegDisplay <= 7'b0100100; end
					4'b0110:begin SegCOM <= 4'b0111;SegDisplay <= 7'b0100000; end
					4'b0111:begin SegCOM <= 4'b0111;SegDisplay <= 7'b0001111; end
					4'b1000:begin SegCOM <= 4'b0111;SegDisplay <= 7'b0000000; end
					4'b1001:begin SegCOM <= 4'b0111;SegDisplay <= 7'b0000100; end
					default:SegCOM = SegCOM;
				endcase
			end
			else if(Count == 4'b0101)begin
				case({ObsVecPosition4} ) //RedDot 4 position
					3'b000:begin COMM <= 4'b1011;DATA_R <= 8'b01111111;DATA_G <= 8'b11111111; end
					3'b001:begin COMM <= 4'b1011;DATA_R <= 8'b10111111;DATA_G <= 8'b11111111; end
					3'b010:begin COMM <= 4'b1011;DATA_R <= 8'b11011111;DATA_G <= 8'b11111111; end
					3'b011:begin COMM <= 4'b1011;DATA_R <= 8'b11101111;DATA_G <= 8'b11111111; end
					3'b100:begin COMM <= 4'b1011;DATA_R <= 8'b11110111;DATA_G <= 8'b11111111; end
					3'b101:begin COMM <= 4'b1011;DATA_R <= 8'b11111011;DATA_G <= 8'b11111111; end
					3'b110:begin COMM <= 4'b1011;DATA_R <= 8'b11111101;DATA_G <= 8'b11111111; end
					3'b111:begin COMM <= 4'b1011;DATA_R <= 8'b11111110;DATA_G <= 8'b11111111; end
					default:COMM = COMM;
				endcase
			end
			else if(Count == 4'b0110)begin
				case({ObsVecPosition5} ) //RedDot 5 position
					3'b000:begin COMM <= 4'b1100;DATA_R <= 8'b01111111;DATA_G <= 8'b11111111; end
					3'b001:begin COMM <= 4'b1100;DATA_R <= 8'b10111111;DATA_G <= 8'b11111111; end
					3'b010:begin COMM <= 4'b1100;DATA_R <= 8'b11011111;DATA_G <= 8'b11111111; end
					3'b011:begin COMM <= 4'b1100;DATA_R <= 8'b11101111;DATA_G <= 8'b11111111; end
					3'b100:begin COMM <= 4'b1100;DATA_R <= 8'b11110111;DATA_G <= 8'b11111111; end
					3'b101:begin COMM <= 4'b1100;DATA_R <= 8'b11111011;DATA_G <= 8'b11111111; end
					3'b110:begin COMM <= 4'b1100;DATA_R <= 8'b11111101;DATA_G <= 8'b11111111; end
					3'b111:begin COMM <= 4'b1100;DATA_R <= 8'b11111110;DATA_G <= 8'b11111111; end
					default:COMM = COMM;
				endcase
			end
			else if(Count == 4'b0111)begin
				case({ObsVecPosition6} ) //RedDot 6 position
					3'b000:begin COMM <= 4'b1101;DATA_R <= 8'b01111111;DATA_G <= 8'b11111111; end
					3'b001:begin COMM <= 4'b1101;DATA_R <= 8'b10111111;DATA_G <= 8'b11111111; end
					3'b010:begin COMM <= 4'b1101;DATA_R <= 8'b11011111;DATA_G <= 8'b11111111; end
					3'b011:begin COMM <= 4'b1101;DATA_R <= 8'b11101111;DATA_G <= 8'b11111111; end
					3'b100:begin COMM <= 4'b1101;DATA_R <= 8'b11110111;DATA_G <= 8'b11111111; end
					3'b101:begin COMM <= 4'b1101;DATA_R <= 8'b11111011;DATA_G <= 8'b11111111; end
					3'b110:begin COMM <= 4'b1101;DATA_R <= 8'b11111101;DATA_G <= 8'b11111111; end
					3'b111:begin COMM <= 4'b1101;DATA_R <= 8'b11111110;DATA_G <= 8'b11111111; end
					default:COMM = COMM;
				endcase
			end
			else if(Count == 4'b1000)begin
				case({ObsVecPosition7} ) //RedDot 7 position
					3'b000:begin COMM <= 4'b1110;DATA_R <= 8'b01111111;DATA_G <= 8'b11111111; end
					3'b001:begin COMM <= 4'b1110;DATA_R <= 8'b10111111;DATA_G <= 8'b11111111; end
					3'b010:begin COMM <= 4'b1110;DATA_R <= 8'b11011111;DATA_G <= 8'b11111111; end
					3'b011:begin COMM <= 4'b1110;DATA_R <= 8'b11101111;DATA_G <= 8'b11111111; end
					3'b100:begin COMM <= 4'b1110;DATA_R <= 8'b11110111;DATA_G <= 8'b11111111; end
					3'b101:begin COMM <= 4'b1110;DATA_R <= 8'b11111011;DATA_G <= 8'b11111111; end
					3'b110:begin COMM <= 4'b1110;DATA_R <= 8'b11111101;DATA_G <= 8'b11111111; end
					3'b111:begin COMM <= 4'b1110;DATA_R <= 8'b11111110;DATA_G <= 8'b11111111; end
					default:COMM = COMM;
				endcase
			end
			else if(Count == 4'b1001)begin
				case({ObsVecPosition8} ) //RedDot 8 position
					3'b000:begin COMM <= 4'b1111;DATA_R <= 8'b01111111;DATA_G <= 8'b11111111; end
					3'b001:begin COMM <= 4'b1111;DATA_R <= 8'b10111111;DATA_G <= 8'b11111111; end
					3'b010:begin COMM <= 4'b1111;DATA_R <= 8'b11011111;DATA_G <= 8'b11111111; end
					3'b011:begin COMM <= 4'b1111;DATA_R <= 8'b11101111;DATA_G <= 8'b11111111; end
					3'b100:begin COMM <= 4'b1111;DATA_R <= 8'b11110111;DATA_G <= 8'b11111111; end
					3'b101:begin COMM <= 4'b1111;DATA_R <= 8'b11111011;DATA_G <= 8'b11111111; end
					3'b110:begin COMM <= 4'b1111;DATA_R <= 8'b11111101;DATA_G <= 8'b11111111; end
					3'b111:begin COMM <= 4'b1111;DATA_R <= 8'b11111110;DATA_G <= 8'b11111111; end
					default:COMM = COMM;
				endcase
			end
		end
		else begin
			DATA_R <= 8'b11111111; //0 is top, 7 is bottom
			DATA_G <= 8'b11111110;
			DATA_B <= 8'b11111111;
		
			SegDisplay <= 7'b0000001;
			SegCOM <= 4'b1110; //0 is current 7segment Displayer || Rightmost is LowerDigit
			COMM <= 4'b1011;
		end
		end
//======================================Create a frequcy to Display======================================
		always @(posedge CLK)begin
			if(Count == 4'b1111)begin  //Sequcially display player,obstacle...
				Count <= 4'b0000;
			end
			else begin
				Count <= Count + 1'b1;
			end
		end
//========================================== Calling Modules =================================================
	KEY_Debounce DebLeft(.CLK(CLK),.KEY_In(LEFT),.KEY_Out(LEFTEDGE));
	KEY_Debounce DebRight(.CLK(CLK),.KEY_In(RIGHT),.KEY_Out(RIGHTEDGE));
	KEY_Debounce DebMiddle(.CLK(CLK),.KEY_In(MIDDLE),.KEY_Out(MIDDLEEDGE));	
	KEY_Debounce DebReset(.CLK(CLK),.KEY_In(RESET),.KEY_Out(RESETEDGE));	
	SpeedRam SpeedRam1(.CLK(CLK),.Speed(Speed1),.SpeedState(SpeedState1),.HARDMODE(HARDMODE));
	SpeedRam SpeedRam2(.CLK(CLK),.Speed(Speed2),.SpeedState(SpeedState2),.HARDMODE(HARDMODE));
	SpeedRam SpeedRam3(.CLK(CLK),.Speed(Speed3),.SpeedState(SpeedState3),.HARDMODE(HARDMODE));
	SpeedRam SpeedRam4(.CLK(CLK),.Speed(Speed4),.SpeedState(SpeedState4),.HARDMODE(HARDMODE));
	SpeedRam SpeedRam5(.CLK(CLK),.Speed(Speed5),.SpeedState(SpeedState5),.HARDMODE(HARDMODE));
	SpeedRam SpeedRam6(.CLK(CLK),.Speed(Speed6),.SpeedState(SpeedState6),.HARDMODE(HARDMODE));
	SpeedRam SpeedRam7(.CLK(CLK),.Speed(Speed7),.SpeedState(SpeedState7),.HARDMODE(HARDMODE));
	SpeedRam SpeedRam8(.CLK(CLK),.Speed(Speed8),.SpeedState(SpeedState8),.HARDMODE(HARDMODE));
	ScoreCount ScoreCountMod(.CLK(CLK),.HARDMODE(HARDMODE),.SCORE(SCORE));

//============================================Obstacle Movement=====================================================================
	always @(posedge CLK) begin
	if(RESETEDGE == 1'b0)begin
	//============================================== ScoreCounting ========================================================
		if(GameOver == 1'b0)begin
		if(SCORE == 1'b1 && HARDMODE == 1'b0)begin
			if(ScoreDigit1 != 4'b 1001)begin
				ScoreDigit1 <= ScoreDigit1 + 1'b1; //Adding Score 1
			end
			else begin
				ScoreDigit1 <= 4'b0000;
				if(ScoreDigit2 != 4'b 1001)begin
					ScoreDigit2 <= ScoreDigit2 + 1'b1;
				end
				else begin
					ScoreDigit2<= 4'b0000;
					if(ScoreDigit3 != 4'b 1001)begin
						ScoreDigit3 <= ScoreDigit3 + 1'b1;
					end
					else begin
						ScoreDigit3<= 4'b0000;
						if(ScoreDigit4 != 4'b 1001)begin
							ScoreDigit4 <= ScoreDigit4 + 1'b1;
						end
						else begin
							GameOver <= 1'b1;
						end
					end
				end
			end
	end
		else if(SCORE == 1'b1 && HARDMODE == 1'b1)begin
		ScoreDigit1 <= ScoreDigit1 + 2'b11; //Adding Score 3
			if(ScoreDigit1 == 4'b1010)begin
				ScoreDigit1 <= 4'b0000;
				if(ScoreDigit2 != 4'b 1001)begin
					ScoreDigit2 <= ScoreDigit2 + 1'b1;
				end
				else begin
					ScoreDigit2<= 4'b0000;
					if(ScoreDigit3 != 4'b 1001)begin
						ScoreDigit3 <= ScoreDigit3 + 1'b1;
					end
					else begin
						ScoreDigit3<= 4'b0000;
						if(ScoreDigit4 != 4'b 1001)begin
							ScoreDigit4 <= ScoreDigit4 + 1'b1;
						end
						else begin
							GameOver <= 1'b1;
						end
					end
				end
			end
			else if(ScoreDigit1 == 4'b1011)begin
				ScoreDigit1 <= 4'b0001;
				if(ScoreDigit2 != 4'b 1001)begin
					ScoreDigit2 <= ScoreDigit2 + 1'b1;
				end
				else begin
					ScoreDigit2<= 4'b0000;
					if(ScoreDigit3 != 4'b 1001)begin
						ScoreDigit3 <= ScoreDigit3 + 1'b1;
					end
					else begin
						ScoreDigit3<= 4'b0000;
						if(ScoreDigit4 != 4'b 1001)begin
							ScoreDigit4 <= ScoreDigit4 + 1'b1;
						end
						else begin
							GameOver <= 1'b1;
						end
					end
				end		
			end
			else if(ScoreDigit1 == 4'b1100)begin
				ScoreDigit1 <= 4'b0010;
				if(ScoreDigit2 != 4'b 1001)begin
					ScoreDigit2 <= ScoreDigit2 + 1'b1;
				end
				else begin
					ScoreDigit2<= 4'b0000;
					if(ScoreDigit3 != 4'b 1001)begin
						ScoreDigit3 <= ScoreDigit3 + 1'b1;
					end
					else begin
						ScoreDigit3<= 4'b0000;
						if(ScoreDigit4 != 4'b 1001)begin
							ScoreDigit4 <= ScoreDigit4 + 1'b1;
						end
						else begin
							GameOver <= 1'b1;
						end
					end
				end		
			end
		end
	end

	//============================================== Moving Part ========================================================	
		if(Speed1 == 1'b1 )begin
		if(Hit1 == 1'b1)begin
			ObsVecPosition1 <= 3'b000;
			Hit1 <= 1'b0;
		end
		else if((ObsVecPosition1 == 3'b110||ObsVecPosition1 == 3'b111)&&Position==3'b000)begin
			GameOver <= 1'b1;
		end
		else if(ObsVecPosition1 == 3'b110&&Position!=3'b000)begin
			ObsVecPosition1 <= ObsVecPosition1 + 1'b1;
			SpeedState1 <= SpeedState1 + 2'b11; // 1,4,2,2
		end
		else begin
			ObsVecPosition1 <= ObsVecPosition1 + 1'b1;
		end
	end
		if(Hit2 == 1'b1)begin
			ObsVecPosition2 <= 3'b000;
			Hit2 <= 1'b0;
		end
		else if(Speed2 == 1'b1 )begin
			if((ObsVecPosition2 == 3'b110||ObsVecPosition2 == 3'b111)&&Position==3'b001)begin
				GameOver <= 1'b1;
			end
			else if(ObsVecPosition1 == 3'b110&&Position!=3'b001)begin
				ObsVecPosition2 <= ObsVecPosition2 + 1'b1;
				SpeedState2 <= SpeedState2 + 2'b01; // 2,2,4,1
			end
			else begin
				ObsVecPosition2 <= ObsVecPosition2 + 1'b1;
			end
		end
		if(Hit3 == 1'b1)begin
			ObsVecPosition3 <= 3'b000;
			Hit3 <= 1'b0;
		end
		else if(Speed3 == 1'b1 )begin
			if((ObsVecPosition3 == 3'b110||ObsVecPosition3 == 3'b111)&&Position==3'b010)begin
				GameOver <= 1'b1;
			end
			else if(ObsVecPosition3 == 3'b110&&Position!=3'b010)begin
				ObsVecPosition3 <= ObsVecPosition3 + 1'b1;
				if(SpeedState3==2'b00)begin
					SpeedState3 <= 2'b01; //1,2,4,2
				end
				else if(SpeedState3==2'b01)begin
					SpeedState3 <= 2'b11; //1,2,4,2
				end
				else if(SpeedState3==2'b11)begin
					SpeedState3 <= 2'b10; //1,2,4,2
				end
				else begin
					SpeedState3 <= 2'b00; //1,2,4,2
				end
			end
			else begin
				ObsVecPosition3 <= ObsVecPosition3 + 1'b1;
			end
		end
		if(Hit4 == 1'b1)begin
			ObsVecPosition4 <= 3'b000;
			Hit4 <= 1'b0;
		end
		else if(Speed4 == 1'b1 )begin
			if((ObsVecPosition4 == 3'b110||ObsVecPosition4 == 3'b111)&&Position==3'b011)begin
				GameOver <= 1'b1;
			end
			else if(ObsVecPosition4 == 3'b110&&Position!=3'b011)begin
				ObsVecPosition4 <= ObsVecPosition4 + 1'b1;
				SpeedState4 <= SpeedState4 + 2'b11; // 2,2,1,4
			end
			else begin
				ObsVecPosition4 <= ObsVecPosition4 + 1'b1;
			end
		end
		//====================================================Obs Above:1,2,3,4 Beneathe 5,6,7,8 
		if(Hit5 == 1'b1)begin
			ObsVecPosition5 <= 3'b000;
			Hit5 <= 1'b0;
		end
		else if(Speed5 == 1'b1 )begin
			if((ObsVecPosition5 == 3'b110||ObsVecPosition5 == 3'b111)&&Position==3'b100)begin
				GameOver <= 1'b1;
			end
			else if(ObsVecPosition5 == 3'b110&&Position!=3'b100)begin
				ObsVecPosition5 <= ObsVecPosition5 + 1'b1;
				SpeedState5 <= SpeedState5 + 2'b01; // 2,4,1,2
			end
			else begin
				ObsVecPosition5 <= ObsVecPosition5 + 1'b1;
			end
		end
		if(Hit6 == 1'b1)begin
			ObsVecPosition6 <= 3'b000;
			Hit6 <= 1'b0;
		end
		else if(Speed6 == 1'b1 )begin
			if((ObsVecPosition6 == 3'b110||ObsVecPosition6 == 3'b111)&&Position==3'b101)begin
				GameOver <= 1'b1;
			end
			else if(ObsVecPosition6 == 3'b110&&Position!=3'b101)begin
				ObsVecPosition6 <= ObsVecPosition6 + 1'b1;
				SpeedState6 <= SpeedState6 + 2'b11; // 2,1,4,2
			end
			else begin
				ObsVecPosition6 <= ObsVecPosition6 + 1'b1;
			end
		end
		if(Hit7 == 1'b1)begin
			ObsVecPosition7 <= 3'b000;
			Hit7 <= 1'b0;
		end
		else if(Speed7 == 1'b1 )begin
			if((ObsVecPosition7 == 3'b110||ObsVecPosition7 == 3'b111)&&Position==3'b110)begin
				GameOver <= 1'b1;
			end
			else if(ObsVecPosition7 == 3'b110&&Position!=3'b110)begin
				ObsVecPosition7 <= ObsVecPosition7 + 1'b1;
				if(SpeedState7==2'b11)begin
					SpeedState7 <= 2'b01; //4,2,1,2
				end
				else if(SpeedState7==2'b01)begin
					SpeedState7 <= 2'b00; //4,2,1,2
				end
				else if(SpeedState7==2'b00)begin
					SpeedState7 <= 2'b10; //4,2,1,2
				end
				else begin
					SpeedState7 <= 2'b11; //4,2,1,2
				end
			end
			else begin
				ObsVecPosition7 <= ObsVecPosition7 + 1'b1;
			end
		end
		if(Hit8 == 1'b1)begin
			ObsVecPosition8 <= 3'b000;
			Hit8 <= 1'b0;
		end
		else if(Speed8 == 1'b1 )begin
		
			if((ObsVecPosition8 == 3'b110||ObsVecPosition8 == 3'b111)&&Position==3'b111)begin
				GameOver <= 1'b1;
			end
			else if(ObsVecPosition8 == 3'b110&&Position!=3'b111)begin
				ObsVecPosition8 <= ObsVecPosition8 + 1'b1;
				SpeedState8 <= SpeedState8 + 2'b01; // 2,2,1,4
			end
			else begin
				ObsVecPosition8 <= ObsVecPosition8 + 1'b1;
			end
		end
	end
	
	else begin
			GameOver <= 1'b0;
			ScoreDigit1 <= 4'b0000; ScoreDigit2 <= 4'b0000;
			ScoreDigit3 <= 4'b0000; ScoreDigit4 <= 4'b0000;
			ammo <= 5'b11110;
			
			ObsVecPosition1 <= 3'b000; ObsVecPosition2 <= 3'b000; ObsVecPosition3 <= 3'b000; ObsVecPosition4 <= 3'b000;
			ObsVecPosition5 <= 3'b000; ObsVecPosition6 <= 3'b000; ObsVecPosition7 <= 3'b000; ObsVecPosition8 <= 3'b000;
			
			SpeedState1 <= SpeedState2;SpeedState2 <= SpeedState3;SpeedState3 <= SpeedState4;SpeedState4 <= SpeedState5;
			SpeedState5 <= SpeedState6;SpeedState6 <= SpeedState7;SpeedState7 <= SpeedState8;SpeedState8 <= 2'b00;	
	end
	//=========================================== Middle Button Reaction ================================================
		if(MIDDLEEDGE == 1'b1 && ammo != 5'b00000)begin
			ammo <= {ammo[3:0], 1'b0};
			case({Position})
				3'b000: Hit1 <= 1'b1;
				3'b001: Hit2 <= 1'b1;
				3'b010: Hit3 <= 1'b1;
				3'b011: Hit4 <= 1'b1;
				3'b100: Hit5 <= 1'b1;
				3'b101: Hit6 <= 1'b1;
				3'b110: Hit7 <= 1'b1;
				3'b111: Hit8 <= 1'b1;
				default:;
			endcase
		end
	end//always' end
	//========================================== Button Reaction Beneathe ===============================================
	//========================================== Handling Player Position =============================================== 
	always @(posedge CLK)begin
			case({LEFTEDGE,RIGHTEDGE})
				2'b10:begin
					if(Position > LEFTLIMIT)begin
						Position <= Position - 1'b1;
					end
					end
				2'b01:begin
					if(Position < RIGHTLIMIT)begin
						Position <= Position + 1'b1;
					end
				end
				default:;
			endcase
			if(RESETEDGE == 1'b1)begin
				Position <= 3'b100;
			end
	end
	
endmodule
