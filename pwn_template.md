```python
#!/usr/bin/env python3

from pwn import context, ELF, process, gdb, cyclic

binary = './'
context.log_level = 'DEBUG'
context.terminal = ['tmux', 'splitw', '-h']
context.binary = binary
elf = ELF(binary)

# to run locally
con = process(binary)
gdb.attach(con, gdbscript='')

# to run on the remote server
# host_and_port = "host.com:123"
# host = host_and_port[:host_and_port.find(':')]
# port = int(host_and_port[host_and_port.find(':')+1:])
# con = remote(host, port)

payload = cyclic(100)

con.sendlineafter(b'', payload)
con.interactive()
```
	