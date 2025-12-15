# Vending_Machine
AIM:

To design, implement, and simulate a Vending Machine Controller using Verilog HDL in Xilinx Vivado, which accepts coins, keeps track of balance, and dispenses products automatically based on predefined pricing using a Finite State Machine (FSM).

APPARATUS / TOOLS REQUIRED:

Computer / Laptop

Vivado (2024.2)

Verilog HDL

PROCEDURE:

First, open Vivado software and create a new RTL project without selecting a default part. Create a new Verilog module named vending_machine.v and define the inputs (clk, reset, coin_5, coin_10) and the output (dispense).

Next, declare the FSM states s0, s5, s10, and s15 to represent the total inserted amount in the machine. The state register stores the current state, and next_state determines the transition based on coin insertion.

Implement the state transition logic using a case statement. On each clock cycle, check which coin (coin_5 or coin_10) is inserted, and update the next_state accordingly. When the total reaches 15 (state s15), the machine transitions back to s0.

The output logic asserts dispense when the current state is s15, indicating that the product should be released.

Finally, create a testbench to simulate different coin combinations, verify the FSM transitions, and observe the dispense signal in Vivado Simulator or GTKWave. Optionally, synthesize the design and generate the bitstream for FPGA implementation.

  VERILOG CODE FOR VENDING MACHINE:
  
    `timescale 1ns / 1ps
        module vending_machine(
          input clk,        // Clock signal
          input reset,      // Synchronous reset signal
          input coin_5,     // Input signal for 5-unit coin
          input coin_10,    // Input signal for 10-unit coin
          output reg dispense // Output signal to dispense product
          );
      
      // FSM states representing total inserted amount
      parameter s0=2'b00, // 0 units inserted
                s5=2'b01, // 5 units inserted
                s10=2'b10, // 10 units inserted
                s15=2'b11; // 15 units inserted (ready to dispense)
      
      reg [1:0] state, next_state; // Current and next state registers
      
      // Sequential block: updates current state on clock edge or reset
      always @(posedge clk or posedge reset)
      begin
          if(reset)
              state <= s0; // On reset, go to initial state (0 units)
          else
              state <= next_state; // Update state to next_state
      end
      
      // Combinational block: determines next state based on current state and coin input
      always @*
      begin
          case(state)
              s0: begin
                  if (coin_5) next_state = s5;   // 5-unit coin inserted → move to s5
                  else if (coin_10) next_state = s10; // 10-unit coin inserted → move to s10
                  else next_state = s0;          // No coin → stay in s0
              end

        s5: begin
            if (coin_5) next_state = s10;  // Add 5-unit coin → total 10 → move to s10
            else if (coin_10) next_state = s15; // Add 10-unit coin → total 15 → move to s15
            else next_state = s5;           // No coin → stay in s5
        end

        s10: begin
            if (coin_5) next_state = s15;  // Add 5-unit coin → total 15 → move to s15
            else if (coin_10) next_state = s15; // Add 10-unit coin → total 20 → treat as 15 for dispense
            else next_state = s10;          // No coin → stay in s10
        end

        s15: begin
            next_state = s0;                // Product dispensed → go back to initial state
        end
    endcase
    end
    
    // Output logic: dispense product when total inserted amount reaches 15 units
    always @*
    begin
        dispense = (state == s15); // dispense is HIGH only in state s15
    end
    
    endmodule


   
      

RTL DIAGRAM:

<img width="1366" height="768" alt="Screenshot (67)" src="https://github.com/user-attachments/assets/e4e3f653-f838-4546-8f30-cda101514378" />

  TESTBENCH FOR VENDING MACHINE:
    
     `timescale 1ns/1ps
      module tb;

        reg clk;
        reg reset;
        reg coin_5;
        reg coin_10;
        wire dispense;
    
        vending_machine uut (
            .clk(clk),
            .reset(reset),
            .coin_5(coin_5),
            .coin_10(coin_10),
            .dispense(dispense)
        );
    
        // Clock generation
        initial clk = 0;
        always #5 clk = ~clk;
    
        initial begin
            reset = 1; coin_5 = 0; coin_10 = 0;
            #20 reset = 0;
    
            // Insert 5 rupee coin
            #10 coin_5 = 1; #10 coin_5 = 0;
    
            // Insert 10 rupee coin → total 15 → dispense
            #20 coin_10 = 1; #10 coin_10 = 0;
    
            #100 $stop;
        end
    
        endmodule

OUTPUT WAVEFORM:

<img width="1366" height="768" alt="waveform" src="https://github.com/user-attachments/assets/0b2d0809-d8e2-4dd9-8ef5-2d90ad01de48" />

CONCLUSION:<br>
The Vending Machine Controller was successfully designed and implemented using Verilog HDL in Xilinx Vivado. The system uses a Finite State Machine (FSM) to track the total amount of coins inserted and controls the dispensing of the product when the required amount is reached.

