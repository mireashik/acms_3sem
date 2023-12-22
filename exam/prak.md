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

### 4. Определить количество и назначение флагов работы FIFO?

```verilog
`define S0 4'd0
`define S1 4'd1
`define S2 4'd2
`define S3 4'd3
`define CR 8'h0D

module UART_Input_Manager #
(
	DIGIT_COUNT = 4
)
(
	input clk,
	input ready_to_load,
	input [7:0] package_to_load,
	output reg [DIGIT_COUNT*4-1 : 0] number_out,
	output reg number_ready
);

reg [2:0] state;
wire [3:0] hex_digit;
reg FIFO_write;
reg FIFO_enable;
wire FIFO_empty;
wire FIFO_full;
wire FIFO_data_out_ready;
wire FIFO_ready_to_load;
wire [3:0] FIFO_data_out;
reg [3:0] temp;

initial 
begin
	state <= `S0;  // Начальное состояние: ожидание
end

always @(posedge clk) 
begin
	case(state)
		`S0: 
			begin 
				number_ready <= 0;
				FIFO_enable <= 0;
				temp <= 0;
				state <= `S1;  // Переход к следующему состоянию
			end
		`S1:
			if (ready_to_load) 
				if (package_to_load == `CR)  // Если принят символ возврата каретки
					if (FIFO_empty)  // Если очередь FIFO пуста
						state <= `S0;  // Вернуться в состояние ожидания
					else
						begin
							FIFO_enable <= 1;  // Включить FIFO
							FIFO_write <= 0;  // Запретить запись в FIFO
							state <= `S3;  // Перейти к состоянию загрузки числа
							number_out <= 0;  // Обнулить выходные данные
						end
				else
					begin
						temp <= hex_digit;  // Сохранить преобразованную цифру
						if (FIFO_full)
							state <= `S0;	// Если FIFO полна, вернуться в состояние ожидания
						else
							begin
								FIFO_enable <= 1;  // Включить FIFO
								FIFO_write <= 1;  // Разрешить запись в FIFO
								state <= `S2;  // Перейти к состоянию ожидания подтверждения записи в FIFO
							end
					end
			else
				FIFO_enable <= 0;  // Если нет готовности к загрузке, выключить FIFO
		`S2:
			if (FIFO_ready_to_load)  // Если FIFO готово к загрузке
				state <= `S1;  // Вернуться к ожиданию новых данных
			else 
				FIFO_enable <= 0;  // Выключить FIFO, если не готово к загрузке
		`S3:
			if (FIFO_empty)  // Если FIFO пусто
				begin
					number_ready <= 1;  // Указать, что число готово
					state <= `S0;  // Вернуться к ожиданию новых данных
				end
			else if (FIFO_data_out_ready)  // Если FIFO готово предоставить данные
				number_out <= {number_out[DIGIT_COUNT*4-5:0],FIFO_data_out};  // Загрузить данные в число
	endcase
end

FIFO fifo(
	.clk(clk),
	.data_in(temp),
	.write(FIFO_write),
	.enable(FIFO_enable),
	.data_out(FIFO_data_out),
	.FIFO_empty(FIFO_empty),
	.FIFO_full(FIFO_full),
	.data_out_ready(FIFO_data_out_ready),
	.ready_to_load(FIFO_ready_to_load)
);

ASCII_To_HEX a1(package_to_load, hex_digit);  // Преобразование ASCII в HEX

endmodule
```
