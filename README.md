module checkpoint2();
  typedef struct packed {
    int time1;
    int core;
    int operation;
    bit [35:0] address;
  } MyStruct;

  MyStruct myArray[$:16];

  // Function to extract information from the address
  function void extractInfo(MyStruct entry);
    bit [1:0] byte_select;
    bit [3:0] column1;
    bit [0:0] channel;
    bit [2:0] bank_group;
    bit [1:0] bank;
    bit [5:0] column2;
    bit [15:0] row;

    // Extract information from the address field
    byte_select = entry.address[1:0];
    column1 = entry.address[5:2];
    channel = entry.address[6:6];
    bank_group = entry.address[9:7];
    bank = entry.address[11:10];
    column2 = entry.address[17:12];
    row = entry.address[33:18];
	// show output at compile time
	`ifdef Debug 
    // Display the extracted information
   $display("Address Info - Byte Select: %0d, Column1: %0d, Channel: %0d, Bank Group: %0d, Bank: %0d, Column2: %0d, Row: %0d",byte_select, column1, channel, bank_group, bank, column2, row); 
//output format
	case(entry.operation)
	0 : begin $display("pre %h %h",bank_group,bank);
	  	  $display("act %h %h %h",bank_group,bank,row);
	          $display("rd %h %h %h",bank_group,bank,column2);
    	    end
	1 : begin $display("pre %h %h",bank_group,bank);
	  	  $display("act %h %h %h",bank_group,bank,row);
	          $display("wr %h %h %h",bank_group,bank,column2);
    	    end 
	2 : begin $display("pre %h %h",bank_group,bank);
	  	  $display("act %h %h %h",bank_group,bank,row);
	          $display("rd %h %h %h",bank_group,bank,column2);
    	    end
	default $display("enter correct operation");
endcase
`endif
  endfunction

  initial begin
int var1;
string var2;
	if ($value$plusargs("%s", var2))
      $display("%s", var2);
    else
      var2 = "checkpoint2.txt";

    var1 = $fopen(var2, "r");
    if (!var1)
      $display("Could not open file: %s", var2);

    // Read data from the file into the array of structures
	while(!$feof(var1)) begin
    for (int i = 0; i < 16 && !$feof(var1); i++) begin
      $fscanf(var1, "%d %d %d %h", myArray[i].time1, myArray[i].core, myArray[i].operation, myArray[i].address);
    end
end
    // Close the file
    $fclose(var1);

    // Display the contents of the array
    for (int i = 0; i < 16; i++) begin
      $display("Entry %0d - Time: %0d, Core: %0d, Operation: %0d, Address: %h", i, myArray[i].time1, myArray[i].core, myArray[i].operation, myArray[i].address);
      // Extract and display information from the address field
      extractInfo(myArray[i]);
    end
  end
endmodule
