# MULTI-BIT-WIDTH BOOTH VECTOR 

**RTL CODE**

```


module Radix4BoothMultiplier #(
    parameter width = 8,  // Width parameter for multiplicand and product
    parameter N = 4       // N parameter for multiplier
) (
    input clk
    
);

    wire signed [width-1:0] multiplicand;
    wire signed [2*N-1:0] multiplier;
    wire signed [2*width-1:0] neg_multiplicand;
    reg [2:0] control_signals [0:N-1];
    reg signed [2*width-1:0] partial_products [0:N-1];
    reg signed [2*width-1:0] sign_extended_products [0:N-1];
    reg signed [2*width-1:0] accumulated_product;
    integer k, i;
    reg [3:0] address;
    
    initial begin
        address =4'b0000;
    end
    
    always @(posedge clk)
    begin 
    address <= address + 4'b0001;
    end
    
blk_mem_gen_0 input1 (
  .clka(clk),    // input wire clka
  .ena(1),      // input wire ena
  .wea(0),      // input wire [0 : 0] wea
  .addra(address),  // input wire [2 : 0] addra
  .dina(dina),    // input wire [7 : 0] dina
  .douta(multiplicand)  // output wire [7 : 0] douta
);

blk_mem_gen_1 input2 (
  .clka(clk),    // input wire clka
  .ena(1),      // input wire ena
  .wea(0),      // input wire [0 : 0] wea
  .addra(address),  // input wire [2 : 0] addra
  .dina(dina),    // input wire [7 : 0] dina
  .douta(multiplier)  // output wire [7 : 0] douta
);

ila_0 multibit (
	.clk(clk), // input wire clk


	.probe0(multiplicand), // input wire [7:0]  probe0  
	.probe1(multiplier), // input wire [7:0]  probe1 
	.probe2(accumulated_product), // input wire [8:0]  probe2 
	.probe3(address) // input wire [3:0]  probe3
);



    // Generate two's complement of multiplicand (M)
    assign neg_multiplicand =  ~multiplicand + 1;
    


    always @ (multiplicand or multiplier or neg_multiplicand) begin
        // Generate control signals
        control_signals[0] = {multiplier[1], multiplier[0], 1'b0}; // Generate C0 for k=0 (special case)
        for (k = 1; k < N; k = k + 1)
            control_signals[k] = {multiplier[2*k+1], multiplier[2*k], multiplier[2*k-1]}; // Generate Ck for each k, for k is not 0

        // Generate partial products
        for (k = 0; k < N; k = k + 1) begin
            case (control_signals[k])
                3'b001, 3'b010 : partial_products[k] = multiplicand;
                3'b011 : partial_products[k] = {multiplicand, 1'b0};
                3'b100 : partial_products[k] = {neg_multiplicand, 1'b0};
                3'b101, 3'b110 : partial_products[k] = neg_multiplicand;
                default : partial_products[k] = 0;
            endcase

            // Sign extend the partial products
            sign_extended_products[k] = $signed(partial_products[k]);
	     
            
            // Multiply by 2 to the power of x or shifting operation
            for (i = 0; i < k; i = i + 1)
                sign_extended_products[k] = {sign_extended_products[k], 2'b00};
            end 
        // Accumulate partial products
        accumulated_product = sign_extended_products[0];
        for (k = 1; k < N; k = k + 1)
            accumulated_product = accumulated_product + sign_extended_products[k]; // Add partial products to get result
	    end

endmodule

```

**TESTBENCH CODE**

```

module Radix4BoothMultiplier_TB;

    // Constants
    parameter width = 8;  // Width parameter for multiplicand and product
    parameter N = 4;      // N parameter for multiplier
    
    // Signals
    reg clk;
    reg signed [width-1:0] multiplicand;
    reg signed [2*N-1:0] multiplier;
    wire signed [2*width-1:0] accumulated_product;
    
    // Instantiate the module under test
    Radix4BoothMultiplier #(
        .width(width),
        .N(N)
    ) dut (
        .clk(clk),
        .multiplicand(multiplicand),
        .multiplier(multiplier),
        .accumulated_product(accumulated_product)
    );
    
    // Clock generation
    initial begin
        clk = 0;
        // Toggle the clock every 5 time units
        forever #5 clk = ~clk;
    end
    
    // Test input values
    initial begin
        // Initialize inputs with test values
        multiplicand =6;   // Change this value as needed
        multiplier = -2;    // Change this value as needed
        
        // Wait for some time to observe the output
        #100;
        
         multiplicand =10;   // Change this value as needed
        multiplier = 1;    // Change this value as needed
        
        // Wait for some time to observe the output
        #100;
        
         multiplicand =-8;   // Change this value as needed
        multiplier = -2;    // Change this value as needed
        
        // Wait for some time to observe the output
        #100;
        
         multiplicand =-9;   // Change this value as needed
        multiplier = 3;    // Change this value as needed
        
        // Wait for some time to observe the output
        #100;
        
        // Display result
        // $display("Product: %d", product);
        
        // End simulation
        $finish;
    end

endmodule

```

**CONSTRAINTS**

```

set_property IOSTANDARD LVCMOS33 [get_ports clk]
set_property PACKAGE_PIN W5 [get_ports clk]


create_clock -period 7.483 -name clk -waveform {0.000 6.000} -add
create_clock -period 7.483 -name clk -waveform {0.000 6.000} [get_ports clk]

```


**COE FILE FOR MULTILICAND**

```

; This .COE file specifies the contents for a block memory
; of depth=8, and width=16.  In this case, values are specified
; in decimal format.
memory_initialization_radix=10;
memory_initialization_vector=
10,
24,
-6,
-11,
-13,
7,
18,
2;

```

**COE FILE FOR MULTIPLIER**

```

; This .COE file specifies the contents for a block memory
; of depth=8, and width=16.  In this case, values are specified
; in decimal format.
memory_initialization_radix=10;
memory_initialization_vector=
-5,
-8,
15,
-20,
-9,
-25,
3,
25;

```
