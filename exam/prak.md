### 1. По приведённому ниже коду на языке Verilog ответьте на вопрос: какое устройство (приёмник или передатчик UART) описывает приведённый код на Verilog HDL и какие параметры имеет данное устройство?

```verilog
// Этот модуль представляет собой приёмник UART (Universal Asynchronous Receiver/Transmitter).
// UART - это стандарт для асинхронной передачи данных между устройствами.
// В данном контексте, этот модуль принимает последовательные биты данных, соответствующие протоколу UART, и преобразует их в параллельные 8-битные данные.

// Приёмник UART (Universal Asynchronous Receiver/Transmitter) – устройство для приёма данных по протоколу UART, который является одним из стандартов для асинхронной последовательной передачи данных между устройствами.
// UART используется для передачи байтов информации, асинхронно, то есть без использования общего тактового сигнала между передатчиком и приёмником.

// Передатчик UART (Universal Asynchronous Receiver/Transmitter) представляет собой устройство, ответственное за передачу данных по асинхронному протоколу UART между двумя устройствами


// Эти строки задают макросы для удобства использования числовых констант в коде
`define RESET 4'd0 // RESET равно 0
`define WAIT_START_BIT 4'd1 // WAIT_START_BIT равно 1
`define LOAD_BIT 4'd2
`define WAIT_HALF_RATE 4'd3

// модуль UART
module UART #(parameter CLOCK_RATE = 100_000_000, parameter BAUD_RATE = 9600) ( // Параметр, определяющий скорость передачи данных по протоколу UART в бодах
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
				state <= `WAIT_START_BIT; // состояние переходит в WAIT_START_BIT.
			end
		
		`WAIT_START_BIT: 
			if (~rx) // Если в состоянии WAIT_START_BIT обнаруживается стартовый бит
				begin
					UART_RX_Idle <= 0; // сигнал UART_RX_Idle устанавливается в 0,
					state <= `LOAD_BIT; // и состояние переходит в LOAD_BIT.
				end
            `LOAD_BIT:
			if (baud_flag) 
				if (bit_counter == 9) // когда считан 9-й бит, проверяется стоп-бит
					begin
						if (rx) // если он корректен, устанавливается флаг UART_RX_Ready_Out
							UART_RX_Ready_Out <= 1; 
						state <= `RESET;
					end				
                        else 
					begin
						if (bit_counter != 0)
							UART_RX_Data_Out <= {rx, UART_RX_Data_Out[7:1]};  // данные добавляются в регистр UART_RX_Data_Out
						bit_counter <= bit_counter + 1; // увеличивается счетчик битов
						state <= `WAIT_HALF_RATE; // состояние переходит в WAIT_HALF_RATE.		
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

### 2. По приведённому ниже коду ответьте на следующие вопросы:
1. Сколько состояний в автомате, управляющем логикой работы устройства?
2. Укажите имена всех объявленных макросов.

#### Ответ выше!

### 3. По приведённому ниже коду на Verilog HDL ответьте на вопрос: является ли рассматриваемый модуль комбинационной схемой или автоматом и почему?
Модуль с использованием всегда-блока (always@(posedge clk)), что указывает на синхронное поведение, а также содержит состояния (reg [1:0] state;).
<br>
Эти признаки говорят о том, что модуль представляет собой последовательную логику и, следовательно, является **автоматом**.

**Автомат** - состояния и синхронное обновление на каждом положительном фронте тактового сигнала.
<br>
Комбинационные схемы - только комбинационную логику без состояний или последовательного поведения.
