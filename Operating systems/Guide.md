
| Operating System, 3 easy Pieces | [Link](file:///Users/rsinghw/Downloads/Operating%20Systems%20-%20Three%20Easy%20Pieces.pdf)         |     |
| ------------------------------- | --------------------------------------------------------------------------------------------------- | --- |
| Easy xv6 setup                  | [Redddit Link](https://garbagecollected.org/2024/08/01/install-xv6/)                                |     |
| Github Assignment               | [Link](https://classroom.github.com/assignment-invitations/1804f02acc555cf5c3a23279052e9cc1/status) |     |
|                                 |                                                                                                     |     |
- After 5 hours We got it.
- The steps to follow to run `xv6` and attach the debugger:
	1. `make CPUS=1 qemu-gdb`
	2. `riscv64-elf-gdb kernel/kernel` (in another window)
	3. `target remote :25501`
	4. This remote target need to be equal, because it the makefile it's dynamic hence it's different every time.