# Verilog Language Learing notes

## Overview

This file was made to recall my verilog language learning notes

## Grammer

### 1. Assign

 `assign`  Consistantly connent a port to another one.

### 2. Wire

 `wire` Act as the line between ports.

### 3. Logical

- 1. `&` bit_And, `&&` Logical_And
- 2. `!` , `~` Not
- 3. `|` bit_or, `||` Logical_Or
- 4. `^` Different

### 4. Vector

`wire [5:0] vec;`
 Vector variable create a group of wires with same characters.

```Verilog
wire [15:0] vec1;
wire [7:0] vec2;
wire [7:0] vec3;

wire [3:-2] vec4;
// Negative range is allowed.

Output [3:0] vec_output;
// default is `wire` type.

assign vec1 = vec2;
//if two vectors have the same size ,it's alright to define like this.
assign vec1 = vec2[7:0];
//valus are set from index [0] , and the lack of value will be set '0'
assign vec1 = {8'b0 , vec2};
//the result is the same as upward. 
```

Use ``default_nettype none` to avoid some hidden bugs.

  - Connect Sign `{}`
  - Repeat Sign `{4{1}}`

### 5.Case

```Verilog
case (sel)
			4'h0: out = a;
			4'h1: out = b;
			4'h2: out = c;
			4'h3: out = d;
			4'h4: out = e;
			4'h5: out = f;
			4'h6: out = g;
			4'h7: out = h;
			4'h8: out = i;
		endcase
```

### 6.Always

```Verilog
 
int m;
always @(*) begin
    for(m=0;m<16;m++)begin

    end
end

```







