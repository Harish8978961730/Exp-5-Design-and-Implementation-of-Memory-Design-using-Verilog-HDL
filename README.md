# Exp-5-Design-and-Simulate the-Memory-Design-using-Verilog-HDL
# Aim
To design and simulate a RAM,ROM,FIFO using Verilog HDL, and verify its functionality through a testbench in the Vivado 2023.1 environment.
Apparatus Required
Vivado 2023.1
Procedure
1. Launch Vivado 2023.1
Open Vivado and create a new project.
2. Design the Verilog Code
Write the Verilog code for the RAM,ROM,FIFO
3. Create the Testbench
Write a testbench to simulate the memory behavior. The testbench should apply various and monitor the corresponding output.
4. Create the Verilog Files
Create both the design module and the testbench in the Vivado project.
5. Run Simulation
Run the behavioral simulation to verify the output.
6. Observe the Waveforms
Analyze the output waveforms in the simulation window, and verify that the correct read and write operation.
7. Save and Document Results
Capture screenshots of the waveform and save the simulation logs. These will be included in the lab report.

# RAM
# Code
```
module ram(
input clk,rst,en,
input [7:0] d,
input [9:0] a,
output reg [7:0] do);
reg [7:0] ram [1023:0];
always@(posedge clk)
begin
 if(rst)
   do <= 0;
 else if(en)
   ram[a] <= d;       
 else
   do <= ram[a];      
end
endmodule
```
 # Test bench
 ```

module ram_tb;
reg clk_t,rst_t,en_t;
reg [7:0] d_t;
reg [9:0] a_t;
wire [7:0] do_t;
ram dut(.clk(clk_t),.rst(rst_t),.en(en_t),.d(d_t),.a(a_t),.do(do_t));
initial
begin
 clk_t=1'b0;
 rst_t=1'b1;
 en_t=1'b0;
 d_t=8'd0;
 a_t=10'd0;
 #50 rst_t=1'b0; 
 en_t=1'b1; a_t=10'd985; d_t=8'd55;
 #40;
 en_t=1'b1;
 a_t=10'd1000;
 d_t=8'd125;
 #40;
 en_t=1'b0;
 a_t=10'd1000;
 #40;
en_t=1'b0;
a_t=10'd985;
 #40;
 $finish;
end
always #10 clk_t = ~clk_t;
endmodule
```
# output Waveform
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/5bf7d566-b3b9-46f7-a8d1-24c98870e991" />

# ROM
 # code
 ```
module rom(
input clk,rst,
input[9:0]a,
output reg [9:0]do );
reg[7:0] rom[1023:0];
initial 
begin
rom[10'd100]=8'd100;
rom[10'd1023]=8'd125;
end
always@(posedge clk)
begin
if(rst)
do<=8'b0;
else
 do<=rom[a];
end
endmodule
```
 
# Test bench
```
module rom_tb;
reg clk_t,rst_t;
reg [9:0] a_t;
wire [7:0] do_t;
rom dut(.clk(clk_t),.rst(rst_t),.a(a_t),.do(do_t));
initial
begin
 clk_t=1'b0;
 rst_t=1'b1;
 a_t=10'd0;
 #50 rst_t=1'b0;
 a_t=10'd100;
 #100;
 a_t=10'd1023;
 #100;
 a_t=10'd200;
 #100;
 $finish;
end
always #10 clk_t=~clk_t;
endmodule
```
# output Waveform
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/652ba9bf-788b-44b3-87ea-41a53e5d948b" />

# FIFO 
# CODE
```
`timescale 1ns / 1ps
module FIFO_SH #(
    parameter Depth = 8,
    parameter Data_width = 8
)(
    input  clk,
    input  rst_n,
    input  w_en,
    input  r_en,
    input  [Data_width-1:0] data_in,
    output reg [Data_width-1:0] data_out,
    output full,
    output empty
);
    reg [$clog2(Depth)-1:0] w_ptr, r_ptr;
    reg [Data_width-1:0] fifo [0:Depth-1];
    integer i;
    always @(posedge clk) begin
        if (!rst_n) begin
            w_ptr    <= 0;
            r_ptr    <= 0;
            data_out <= 0;
            for (i = 0; i < Depth; i = i + 1)
                fifo[i] <= 0;
        end
    end
    always @(posedge clk) begin
        if (w_en && !full) begin
            fifo[w_ptr] <= data_in;
            w_ptr <= (w_ptr + 1) % Depth;
        end
    end
    always @(posedge clk) begin
        if (r_en && !empty) begin
            data_out <= fifo[r_ptr];
            r_ptr <= (r_ptr + 1) % Depth;
        end
    end
    assign full  = ((w_ptr + 1) % Depth) == r_ptr;
    assign empty = (w_ptr == r_ptr);
endmodule
```
# TEST BENCH
```
`timescale 1ns/1ps
module FIFO_SH_tb;
    parameter Depth = 8;
    parameter Data_width = 8;
    reg clk_t, rst_n_t;
    reg w_en_t, r_en_t;
    reg [Data_width-1:0] data_in_t;
    wire [Data_width-1:0] data_out_t;
    wire full_t, empty_t;
    FIFO_SH #(.Depth(Depth), .Data_width(Data_width)) uut (
        .clk(clk_t),
        .rst_n(rst_n_t),
        .w_en(w_en_t),
        .r_en(r_en_t),
        .data_in(data_in_t),
        .data_out(data_out_t),
        .full(full_t),
        .empty(empty_t)
    );
    initial begin
        clk_t = 0;
        forever #10 clk_t = ~clk_t; // 20ns period
    end
    initial begin
        // Initialization
        rst_n_t = 0; w_en_t = 0; r_en_t = 0; data_in_t = 0;
        #50;
        rst_n_t = 1;
        #20; w_en_t = 1; data_in_t = 8'd50;
        #20; data_in_t = 8'd40;
        #20; data_in_t = 8'd30;
        #20; data_in_t = 8'd20;
        #20; w_en_t = 0;
        #40; r_en_t = 1;
        repeat (4) begin
            #20;
            $display("Time=%0t | Read Data = %0d", $time, data_out_t);
        end
        #20; r_en_t = 0;
        #100 $stop;
    end
endmodule
```
# OUTPUT WAVEFORM 
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a8c21590-5f12-4259-8f8a-46593e7149ce" />

# Conclusion
The RAM, ROM and FIFO memory with read and write operations was designed and successfully simulated using Verilog HDL. The testbench verified both the write and read functionalities by simulating the memory operations and observing the output waveforms. The experiment demonstrates how to implement memory operations in Verilog, effectively modeling both the reading and writing processes.
 
 

