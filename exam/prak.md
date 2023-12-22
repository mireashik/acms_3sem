1. По приведённому ниже коду на языке Verilog ответьте на вопрос: какое устройство (приёмник или передатчик UART) описывает приведённый код на Verilog HDL и какие параметры имеет данное устройство?

```verilog
// Эти строки задают макросы для удобства использования числовых констант в коде
`define RESET 4'd0 // RESET равно 0
`define WAIT_START_BIT 4'd1 // WAIT_START_BIT равно 1
`define LOAD_BIT 4'd2
`define WAIT_HALF_RATE 4'd3

// модуль UART
module UART #(parameter CLOCK_RATE = 100_000_000, parameter BAUD_RATE = 9600) (
	input clk, // тактовый сигнал
	input rx, // входные последовательные данные
	output reg UART_RX_Idle,
	output reg UART_RX_Ready_Out,
	output reg [7:0] UART_RX_Data_Out
);

reg [1:0] state; // Здесь объявляются регистры для хранения состояния конечного автомата
reg [3:0] bit_counter; // счетчика битов
reg [$clog2(CLOCK_RATE / BAUD_RATE):0] baud_counter; // счетчика для генерации битов передачи 
reg baud_flag; // флага для управления сэмплированием входных данных

initial 
begin
	state = `RESET;
	baud_flag = 0;
end

always@(posedge clk)
begin
	// определить логику для каждого состояния конечного автомата
	case(state)
		//  состоянии RESET устанавливаются начальные значения,
		`RESET: 
			begin
				bit_counter <= 0; 	
				baud_counter <= 0;
				UART_RX_Idle <= 1;
				UART_RX_Data_Out <= 0;
				UART_RX_Ready_Out <= 0;
				state <= `WAIT_START_BIT;
			end
		`WAIT_START_BIT: 
			if (~rx)
				begin
					UART_RX_Idle <= 0;
					state <= `LOAD_BIT;
				end
            `LOAD_BIT:
			if (baud_flag) 
				if (bit_counter == 9)
					begin
						if (rx) 
							UART_RX_Ready_Out <= 1; 
						state <= `RESET;
					end				
                        else 
					begin
						if (bit_counter != 0)
							UART_RX_Data_Out <= {rx, UART_RX_Data_Out[7:1]};  // Загрузка бита данных
						bit_counter <= bit_counter + 1; 
						state <= `WAIT_HALF_RATE;		
					end	
		`WAIT_HALF_RATE:
			if (baud_flag)
				state <= `LOAD_BIT;	
	endcase
end

if (baud_counter == CLOCK_RATE / BAUD_RATE / 2)
	begin
		baud_flag <= 1;  // Установка флага для сэмплирования входных данных
		baud_counter <= 0;
	end
else 
	begin
		baud_flag <= 0;		
		baud_counter <= baud_counter + 1;
	end

endmodule
```
