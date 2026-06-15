# PROJECT 1-SOFTMAX-ATTENTION-ON-FPGA-DE10
- Khối module top attention_top_with_uart
<img width="1764" height="1529" alt="module top attention_top_with_uart" src="https://github.com/user-attachments/assets/54c09d1c-d87e-4464-85ce-2ba03f814eec" />
- Khối module phase 1: Linear; phase 2: attention score; phase 3: computing softmax
<img width="1899" height="1316" alt="module linear softmax" src="https://github.com/user-attachments/assets/3754db3e-01a1-4793-8f80-e72c86d33eb7" />
# Danh sách tín hiệu I/O các module

## Mục lục
- [uart_rx](#uart_rx)
- [uart_tx](#uart_tx)
- [uart_controller](#uart_controller)
- [ram_1port](#ram_1port)
- [exp_rom](#exp_rom)
- [linear_multiply_accumulate](#linear_multiply_accumulate)
- [attention_score_calc](#attention_score_calc)
- [linear_top](#linear_top)
- [softmax_findmax_sub](#softmax_findmax_sub)
- [softmax_exp_sum](#softmax_exp_sum)
- [softmax_divider](#softmax_divider)
- [attention_top](#attention_top)
- [attention_top_with_uart](#attention_top_with_uart)

---

## uart_rx

File: `uart_rx.sv`

| Tín hiệu | Hướng | Kiểu/Độ rộng | Ghi chú |
|---|---|---|---|
| i_clk | input | logic | Clock hệ thống |
| i_rst_n | input | logic | Reset, active-low |
| i_rx | input | logic | Chân RX nối FT232R |
| o_rx_data | output | logic [7:0] | Byte dữ liệu nhận được |
| o_rx_valid | output | logic | Pulse 1 clock = byte sẵn sàng |

---

## uart_tx

File: `uart_tx.sv`

| Tín hiệu | Hướng | Kiểu/Độ rộng | Ghi chú |
|---|---|---|---|
| i_clk | input | logic | Clock hệ thống |
| i_rst_n | input | logic | Reset, active-low |
| i_tx_data | input | logic [7:0] | Byte cần gửi |
| i_tx_start | input | logic | Pulse 1 clock để bắt đầu |
| o_tx | output | logic | Chân TX nối FT232R |
| o_tx_busy | output | logic | =1 khi đang gửi |

---

## uart_controller

File: `uart_controller.sv`

| Tín hiệu | Hướng | Kiểu/Độ rộng | Ghi chú |
|---|---|---|---|
| i_clk | input | logic | Clock |
| i_rst_n | input | logic | Reset |
| i_rx_data | input | logic [7:0] | Từ uart_rx |
| i_rx_valid | input | logic | Từ uart_rx |
| o_tx_data | output | logic [7:0] | Tới uart_tx |
| o_tx_start | output | logic | Tới uart_tx |
| i_tx_busy | input | logic | Từ uart_tx |
| o_rx_ram_we | output | logic | Ghi RAM đệm X |
| o_rx_ram_addr | output | logic [7:0] | Địa chỉ word (0..191) |
| o_rx_ram_data | output | logic [15:0] | Dữ liệu word X |
| o_start_linear | output | logic | Điều khiển attention_top |
| o_token_sel | output | logic [1:0] | Chọn token |
| i_linear_done | input | logic | Từ attention_top |
| o_start_attn | output | logic | Điều khiển attention_top |
| i_attn_done | input | logic | Từ attention_top |
| o_start_softmax | output | logic | Điều khiển attention_top |
| i_softmax_done | input | logic | Từ attention_top |
| i_y_we | input | logic | Ghi kết quả Y |
| i_y_addr | input | logic [3:0] | Địa chỉ Y |
| i_y_data | input | logic signed [15:0] | Giá trị Y |
| o_result_ready | output | logic | Debug |
| o_state_debug | output | logic [3:0] | Debug, 0..12 |

---

## ram_1port

File: `attention_top.sv`

| Tín hiệu | Hướng | Kiểu/Độ rộng | Ghi chú |
|---|---|---|---|
| clk | input | logic | Clock |
| we | input | logic | Write enable |
| addr | input | logic [ADDR_WIDTH-1:0] | Địa chỉ |
| data_in | input | logic signed [DATA_WIDTH-1:0] | Dữ liệu ghi |
| data_out | output | logic signed [DATA_WIDTH-1:0] | Dữ liệu đọc (đồng bộ) |

---

## exp_rom

File: `attention_top.sv`

| Tín hiệu | Hướng | Kiểu/Độ rộng | Ghi chú |
|---|---|---|---|
| address | input | logic [10:0] | Địa chỉ tra cứu |
| clock | input | logic | Clock |
| q | output | logic [15:0] | Giá trị e^Z |

---

## linear_multiply_accumulate

File: `attention_top.sv`

| Tín hiệu | Hướng | Kiểu/Độ rộng | Ghi chú |
|---|---|---|---|
| i_clk | input | logic | Clock |
| i_reset_n | input | logic | Reset |
| i_start_processing | input | logic | Bắt đầu tính |
| i_token_index | input | logic [1:0] | Chỉ số token |
| o_processing_done | output | logic | Hoàn tất |
| o_ram_we | output | logic | Ghi RAM Q/K/V |
| o_ram_write_addr | output | logic [7:0] | Địa chỉ ghi |
| i_input_data_x | input | logic signed [DATA_WIDTH-1:0] | Dữ liệu X từ ROM |
| i_weight_query_data | input | logic signed [DATA_WIDTH-1:0] | Trọng số Wq |
| i_weight_key_data | input | logic signed [DATA_WIDTH-1:0] | Trọng số Wk |
| i_weight_value_data | input | logic signed [DATA_WIDTH-1:0] | Trọng số Wv |
| o_input_x_address | output | logic [7:0] | Địa chỉ đọc X |
| o_weight_matrix_address | output | logic [11:0] | Địa chỉ đọc ma trận trọng số |
| o_query_result | output | logic signed [DATA_WIDTH-1:0] | Kết quả Q |
| o_key_result | output | logic signed [DATA_WIDTH-1:0] | Kết quả K |
| o_value_result | output | logic signed [DATA_WIDTH-1:0] | Kết quả V |

---

## attention_score_calc

File: `attention_top.sv`

| Tín hiệu | Hướng | Kiểu/Độ rộng | Ghi chú |
|---|---|---|---|
| clk | input | logic | Clock |
| rst_n | input | logic | Reset |
| start_calc | input | logic | Bắt đầu tính |
| o_rd_ext_mode | output | logic | Chế độ đọc ngoài |
| o_q_addr | output | logic [7:0] | Địa chỉ đọc Q |
| o_k_addr | output | logic [7:0] | Địa chỉ đọc K |
| i_q_data | input | logic signed [15:0] | Dữ liệu Q |
| i_k_data | input | logic signed [15:0] | Dữ liệu K |
| o_s_ram_we | output | logic | Ghi RAM S |
| o_s_ram_addr | output | logic [3:0] | Địa chỉ S |
| o_s_ram_data | output | logic signed [31:0] | Giá trị S = Q·Kᵀ/scale |
| done_calc | output | logic | Hoàn tất |

---

## linear_top

File: `attention_top.sv`

| Tín hiệu | Hướng | Kiểu/Độ rộng | Ghi chú |
|---|---|---|---|
| i_clock | input | logic | Clock |
| i_reset_n | input | logic | Reset |
| i_rx_ram_we | input | logic | Ghi RAM X từ UART |
| i_rx_ram_addr | input | logic [7:0] | Địa chỉ X |
| i_rx_ram_data | input | logic signed [15:0] | Dữ liệu X |
| i_wq_we | input | logic | Ghi RAM Wq |
| i_wq_addr | input | logic [11:0] | Địa chỉ Wq |
| i_wq_data | input | logic [15:0] | Dữ liệu Wq |
| i_wk_we | input | logic | Ghi RAM Wk |
| i_wk_addr | input | logic [11:0] | Địa chỉ Wk |
| i_wk_data | input | logic [15:0] | Dữ liệu Wk |
| i_wv_we | input | logic | Ghi RAM Wv |
| i_wv_addr | input | logic [11:0] | Địa chỉ Wv |
| i_wv_data | input | logic [15:0] | Dữ liệu Wv |
| i_start_linear | input | logic | Bắt đầu phase Linear |
| i_token_selection | input | logic [1:0] | Chọn token |
| o_linear_done | output | logic | Hoàn tất Linear |
| i_start_attn | input | logic | Bắt đầu phase Attention |
| o_attn_done | output | logic | Hoàn tất Attention |
| o_s_ram_we | output | logic | Ghi RAM S |
| o_s_ram_addr | output | logic [3:0] | Địa chỉ S |
| o_s_ram_data | output | logic signed [31:0] | Giá trị S |

---

## softmax_findmax_sub

File: `attention_top.sv`

| Tín hiệu | Hướng | Kiểu/Độ rộng | Ghi chú |
|---|---|---|---|
| clk | input | logic | Clock |
| rst_n | input | logic | Reset |
| i_s_ram_we | input | logic | Ghi S vào S_RAM nội bộ |
| i_s_ram_wr_addr | input | logic [3:0] | Địa chỉ ghi S |
| i_s_ram_wr_data | input | logic signed [31:0] | Dữ liệu S |
| start_softmax | input | logic | Bắt đầu |
| done_softmax | output | logic | Hoàn tất |
| o_z_ram_we | output | logic | Ghi Z_INTER_RAM |
| o_z_ram_wr_addr | output | logic [3:0] | Địa chỉ Z |
| o_z_ram_data | output | logic signed [DATA_WIDTH-1:0] | Giá trị Z = X - Max |

---

## softmax_exp_sum

File: `attention_top.sv`

| Tín hiệu | Hướng | Kiểu/Độ rộng | Ghi chú |
|---|---|---|---|
| clk | input | logic | Clock |
| rst_n | input | logic | Reset |
| start_exp_sum | input | logic | Bắt đầu |
| done_exp_sum | output | logic | Hoàn tất |
| o_s_ram_rd_addr | output | logic [3:0] | Địa chỉ đọc Z_INTER_RAM |
| i_s_ram_rd_data | input | logic signed [15:0] | Dữ liệu Z |
| o_z_ram_we | output | logic | Ghi Z_FINAL_RAM |
| o_z_ram_wr_addr | output | logic [3:0] | Địa chỉ ghi Exp(Z) |
| o_z_ram_data | output | logic signed [DATA_WIDTH-1:0] | Giá trị Exp(Z) |
| o_sum_exp_q88 | output | logic [23:0] [0:SEQ_LEN-1] | Tổng Σ Exp(Z), mảng theo token |

---

## softmax_divider

File: `attention_top.sv`

| Tín hiệu | Hướng | Kiểu/Độ rộng | Ghi chú |
|---|---|---|---|
| clk | input | logic | Clock |
| rst_n | input | logic | Reset |
| start_div | input | logic | Bắt đầu |
| done_div | output | logic | Hoàn tất |
| i_sum_exp | input | logic [23:0] [0:SEQ_LEN-1] | Mẫu số (Σ Exp), mảng |
| o_z_ram_rd_addr | output | logic [3:0] | Địa chỉ đọc Z_FINAL_RAM |
| i_z_ram_rd_data | input | logic signed [15:0] | Dữ liệu Exp(Z) |
| o_y_ram_we | output | logic | Ghi kết quả Y |
| o_y_ram_wr_addr | output | logic [3:0] | Địa chỉ Y |
| o_y_ram_data | output | logic signed [15:0] | Giá trị Y (attention weight) |

---

## attention_top

File: `attention_top.sv` (top-level)

| Tín hiệu | Hướng | Kiểu/Độ rộng | Ghi chú |
|---|---|---|---|
| clk | input | logic | Clock |
| rst_n | input | logic | Reset |
| i_start_linear | input | logic | Bắt đầu Phase 1&2 |
| i_token_selection | input | logic [1:0] | Chọn token |
| o_linear_done | output | logic | Hoàn tất Linear |
| i_start_attn | input | logic | Bắt đầu Attention |
| o_attn_done | output | logic | Hoàn tất Attention |
| i_start_softmax | input | logic | Bắt đầu Phase 3 |
| o_softmax_done | output | logic | Hoàn tất Softmax |
| o_softmax_sum | output | logic [23:0] [0:SEQ_LEN-1] | Tổng Σ Exp(Z) |
| i_rx_ram_we | input | logic | Ghi RAM X (từ UART) |
| i_rx_ram_addr | input | logic [7:0] | Địa chỉ X |
| i_rx_ram_data | input | logic signed [15:0] | Dữ liệu X |
| i_wq_we | input | logic | Ghi RAM Wq |
| i_wq_addr | input | logic [11:0] | Địa chỉ Wq |
| i_wq_data | input | logic [15:0] | Dữ liệu Wq |
| i_wk_we | input | logic | Ghi RAM Wk |
| i_wk_addr | input | logic [11:0] | Địa chỉ Wk |
| i_wk_data | input | logic [15:0] | Dữ liệu Wk |
| i_wv_we | input | logic | Ghi RAM Wv |
| i_wv_addr | input | logic [11:0] | Địa chỉ Wv |
| i_wv_data | input | logic [15:0] | Dữ liệu Wv |
| o_z_ram_we | output | logic | Ghi kết quả Y cuối |
| o_z_ram_wr_addr | output | logic [3:0] | Địa chỉ Y |
| o_z_ram_data | output | logic signed [DATA_WIDTH-1:0] | Giá trị Y |

---

## attention_top_with_uart

File: `attention_top_with_uart.sv` (top-level cho DE10-Standard)

| Tín hiệu | Hướng | Kiểu/Độ rộng | Ghi chú |
|---|---|---|---|
| CLOCK_50 | input | logic | Clock 50MHz |
| KEY | input | logic [1:0] | KEY[0]=reset, KEY[1]=chuyển hàng HEX |
| UART_RXD | input | logic | UART nhận |
| UART_TXD | output | logic | UART gửi |
| o_result_ready | output | logic | Báo kết quả sẵn sàng |
| HEX0 | output | logic [6:0] | 7-đoạn (active-low) |
| HEX1 | output | logic [6:0] | 7-đoạn |
| HEX2 | output | logic [6:0] | 7-đoạn |
| HEX3 | output | logic [6:0] | 7-đoạn |
| HEX4 | output | logic [6:0] | 7-đoạn |
| HEX5 | output | logic [6:0] | 7-đoạn |
