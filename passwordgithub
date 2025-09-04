`timescale 1ns / 1ps
//////////////////////////////////////////////////////////////////////////////////
// Company: 
// Engineer: 
// 
// Create Date: 2025/08/29 19:37:15
// Design Name: 
// Module Name: PassWord
// Project Name: 
// Target Devices: 
// Tool Versions: 
// Description: 
// 
// Dependencies: 
// 
// Revision:
// Revision 0.01 - File Created
// Additional Comments:
// 
//////////////////////////////////////////////////////////////////////////////////
//工作部分流程  
//

module PassWord(
    input key_change,      //该按键推上后进入更改密码的状态
    input key_read,        //该按键按下后进入读密码的状态
    input [9:0]key_input,  //当key_read为1将其中为1的值（高电平）读入密码
    input key_delete,      //该按键按下后回退到上一个状态（即删除一位密码）
    input key_enter,       //确定键
    input key_restart,     //管理员专用按键（当进入报警状态时按下可以回到等待状态）
    input clk_100MHz,
    output reg key_right=0,      //密码正确
    output reg key_wrong=0,      //密码错误
    output reg key_warn,        //连续三次密码错误进入报警状态
    output reg key_change_state=0,//为1则亮起象征改变状态的灯
    output reg key_normal_state=0 //为1则亮起象征正常状态的灯
);
    parameter PASSWORD_LENGTH =4;    
    reg  [3:0]stored_password[0:PASSWORD_LENGTH-1]='{default:4'd0};//存储的密码值（初省值各元素均为0，修改密码时可以直接修改）
    reg  [3:0]input_password[0:PASSWORD_LENGTH-1]='{default:4'd0}; //密码锁工作时输入的密码值（正常工作后续会和存储的密码比较）
    //消抖后按钮确实按下的对应寄存器需要有值的变化
    reg key_read_r=1'b0;//等于1表示进入读密码状态(已消抖)   
    reg key_delete_r=1'b0;//等于1表示应执行删除密码操作(已消抖)
    reg key_enter_r=1'b0;    //等于1表示确定
    reg key_restart_r=1'b0;    //等于1表示重启
    reg key_state=1'd0;//为1正在输入第一位，为2第二位，以此类推
    reg count_10s=1'b0 ;  //10s计时器  为1计满
    reg count_10s_start=1'b0;//10s计时器  为1开始计时
    reg count_20s=1'b0;   //20s计时器  为1计满
    reg count_20s_start=1'b0;//20s计时器  为1开始计时
    reg wrong_times=1'd0;//犯错次数
    //消抖


    // 按键编码器
    function automatic [3:0] key_encoder(input [9:0] keys);
        begin
            case(keys)
                10'b0000_0000_01: key_encoder = 4'd0;  // 按键0
                10'b0000_0000_10: key_encoder = 4'd1;  // 按键1
                10'b0000_0001_00: key_encoder = 4'd2;  // 按键2
                10'b0000_0010_00: key_encoder = 4'd3;  // 按键3
                10'b0000_0100_00: key_encoder = 4'd4;  // 按键4
                10'b0000_1000_00: key_encoder = 4'd5;  // 按键5
                10'b0001_0000_00: key_encoder = 4'd6;  // 按键6
                10'b0010_0000_00: key_encoder = 4'd7;  // 按键7
                10'b0100_0000_00: key_encoder = 4'd8;  // 按键8
                10'b1000_0000_00: key_encoder = 4'd9;  // 按键9
                default: key_encoder = 4'hF;           // 无效输入
            endcase
        end
    endfunction
   

    //正常输密码输入第一位
    always @(posedge clk_100MHz&&key_state==0)begin
        if(key_change==0)begin
            if (key_read_r) begin
                input_password[key_state]<= key_encoder(key_input);
                key_state++;
            end
            else;
        end
        //改密码
        else begin
                if (key_read_r) begin
                    stored_password[key_state]<= key_encoder(key_input);
                    key_state++;
                end
                else;
            end
    end
    //输入第二位
    always @(posedge clk_100MHz&&key_state==1)begin
        if(key_change==0)begin
            count_10s_start=1;
            #20;
            count_10s_start=0;
            if(key_delete_r)begin
                key_state<=0;
                input_password[0]<=0;
            end
            else if (key_read_r) begin
                input_password[key_state]<= key_encoder(key_input);
                key_state++;
            end
            else if (count_10s) begin
                key_state<=0;
            end
            else;
        end
        //改密码
        else begin
            count_10s_start=1;
            #20;
            count_10s_start=0;
            if(key_delete_r)begin
                key_state<=0;
                stored_password[0]<=0;
            end
            else if (key_read_r) begin
                stored_password[key_state]= key_encoder(key_input);
                key_state++;
            end
            else if (count_10s) begin
                key_state<=0;
            end
            else;
        end 
    end
    //输入第三位
    always @(posedge clk_100MHz&&key_state==2)begin
        if(key_change==0)begin
            count_10s_start=1;
            #20;
            count_10s_start=0;
            if(key_delete_r)begin
                key_state<=1;
                input_password[1]<=0;
            end
            else if (key_read_r) begin
                input_password[key_state]<= key_encoder(key_input);
                key_state++;
            end
            else if (count_10s) begin
                key_state<=0;
            end
            else;
        end
        //改密码
        else begin
            count_10s_start=1;
            #20;
            count_10s_start=0;
            if(key_delete_r)begin
                key_state<=1;
                stored_password[1]<=0;
            end
            else if (key_read_r) begin
                stored_password[key_state]= key_encoder(key_input);
                key_state++;
            end
            else if (count_10s) begin
                key_state<=0;
            end
            else;
        end 
    end
    //输入第四位
    always @(posedge clk_100MHz&&key_state==3)begin
        if(key_change==0)begin
            count_10s_start=1;
            #20;
            count_10s_start=0;
            if(key_delete_r)begin
                key_state<=2;
                input_password[2]<=0;
            end
            else if (key_read_r) begin
                input_password[key_state]<= key_encoder(key_input);
                key_state++;
            end
            else if (count_10s) begin
                key_state<=0;
            end
            else;
        end
        //改密码
        else begin
            count_10s_start=1;
            #20;
            count_10s_start=0;
            if(key_delete_r)begin
                key_state<=2;
                stored_password[2]<=0;
            end
            else if (key_read_r) begin
                stored_password[key_state]= key_encoder(key_input);
                key_state++;
            end
            else if (count_10s) begin
                key_state<=0;
            end
            else;
        end 
    end
    //密码输入完毕后确认比较
    always @(posedge clk_100MHz&&key_state==4)begin 
        if(wrong_times>=3) key_warn<=1;//警告状态
        else begin
            if (key_change==0) begin
                count_10s_start=1;
                #20;
                count_10s_start=0;
                if(key_delete_r)begin
                    key_state<=3;
                    input_password[3]<=0;
                end
                else if (key_enter_r) begin
                    if (input_password==stored_password) begin
                        key_right<=1;//正确
                        #100;
                        count_20s_start=1;
                        #20;
                        count_20s_start=0;
                        if (key_enter_r) begin
                            key_state<=0;
                            key_right<=0;
                        end
                        else if (count_20s) begin
                            key_state<=0;
                            count_20s<=0;
                            key_right<=0;
                        end
                        else;
                    end
                    else begin
                        key_wrong<=1;
                        #2000;
                        wrong_times<=wrong_times+1;
                        key_state<=0;
                        key_wrong<=0;
                    end
                end
                else if (count_10s) begin
                    key_state<=0;
                end
                else;
            end
        else begin
            key_state<=0;
        end
        end
    end

    //用灯的亮来表示不同状态

    always @(posedge clk_100MHz) begin
        if (key_change) begin
            key_change_state<=1;
            key_normal_state<=0;
        end
        else begin 
            key_normal_state<=1;
            key_change_state<=0;
        end
    end
    always @(posedge clk_100MHz&&key_warn==1) begin
        if (key_restart) begin
            key_state<=0;
        end
        else;
    end

    always @(posedge clk_100MHz&&count_10s_start==1) begin
        
    end//计时10s

endmodule
