Some notes on setup the environment are [here](https://github.com/eecsmap/logs/blob/main/eecs/ucb_eecs151.md)

###### Interface of FIFO

The Ready-Valid protocol can indeed be somewhat loose in certain scenarios because it allows both parties to have a degree of control over the data transfer, which might make it difficult to clearly specify who should initiate the communication.

For devices like FIFOs, designers often prefer to have a clear initiator in the communication process, which is typically the data producer or consumer. The FIFO user can query the FIFO's status using empty and full signals, which makes it more intuitive for the user to initiate the communication. The FIFO can provide its status, and the user (such as the data producer or consumer) can decide when to start the data transfer based on these status signals.

So for the writing side, we can have:
* input: data_in
* input: write_enable
* output: is_full

And for the reading side, we can have:
* input: read_enable
* output: data_out
* output: is_empty

# FPGA Project

Following notes are based on the [2022 fall FPGA project](https://github.com/eecsmap/fpga_project_skeleton_fa22).
