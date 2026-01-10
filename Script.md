First things first there are some bugs in the code that i showed you guys in the previous videos
so lets fix them first
i was able to find these bugs after i setup nvim for rust since we cant debug this code using IDEs as it doesnt compile on windows so i setup neovim in wsl
you can find my nvim config at
https://github.com/sharmachait/nvim-config
any ways so the bugs are in the read_all_registers function
where we are calling the ptrace sys call with PTRACE_GETFPREGS
we passed in fpregs as non mut const
but it needs to be mutable
![[{B91A33C0-3541-4839-B305-855B3611B6F3}.png]]
we have the same bug in read_all_registers of process as well which we will fix when we get to it
the next bug is in the write_registers function in process_registers, not really a bug but someething we should be validating when we are taking user input
![[{9AD72B46-D6A2-4E89-AE2A-0A4908F8DBA5}.png]]
![[{04DE31AB-C757-4234-B2B7-7295017F7238}.png]]

now the biggest bug that we have if were to see the get_register_val function in process_registers
for LongDouble we are trying to simply cast bytes to f128 object but since its not a built in data type this doesnt work

we need to do handle that manually
this wouldnt have been an issue in c++ but since we are above coding in c++ we will do it manually

this f128 data type belongs to some stack register

so we will begin by determining the index of stack register

since each register is 16bytes that is 4 u32s per st register we can get the actual offset by multiplying the index by 4

we want to read the data from this offset so we will read 16 bytes
since our user object has u32s for st_space
https://dev.azure.com/ChaitanyaDSharma/Codecrafters/_git/rdb?path=/src/rdb/register_info.rs&version=GBmaster&line=213&lineEnd=214&lineStartColumn=1&lineEndColumn=1&lineStyle=plain&_a=contents

to read we will read a u32 and that will be converted to 4 u8s
and then we will create a helper method that will take an array of bytes and create a float out of it
![[{949A6EA2-FB37-44BA-B8E1-B41352B4C894}.png]]
we also need to do the vice versa of this in write_register function
![[{067D8723-39F2-4C29-B300-CCB46791F0A5}.png]]
![[{B8CAE024-9716-47D3-AB33-121D774930F7}.png]]
![[{A081C1B2-A756-4636-A4CC-2B0268803F1E}.png]]

we also need to update widen function
replace 
```rust
RegisterValue::to_byte128(val)
```
with
```rust
let f64_val = val.to_string().parse::<f64>().unwrap_or(0.0);
return RegisterValue::f64_to_x87_bytes(f64_val);
```
we need to do this because the LongDouble isnt actually 128 bits it is 80 bits or 8 bytes 

so maybe later on i will refactor this code and remove f128 and do arrays of bytes instead

we now need to write the helper functions that we have used in get_register_val and widen

```rust
impl RegisterValue {
fn f64_to_x87_bytes(val: f64) -> [u8; 16] {
        let mut result = [0u8; 16];
  
        if val == 0.0 {
            return result;
        }
  
        let bits = val.to_bits();
        let sign = (bits >> 63) & 1;
        let exp = ((bits >> 52) & 0x7FF) as i32;
        let mantissa = bits & 0xFFFFFFFFFFFFF;
  
        if exp == 0 {
            return result;
        }
  
        if exp == 0x7FF {
            return result;
        }
  
        let x87_exp = (exp - 1023 + 16383) as u16;
        let x87_mantissa = (1u64 << 63) | (mantissa << 11);
  
        // Debug output
        println!("DEBUG f64_to_x87: val={}, mantissa=0x{:016x}, exp={}, x87_exp=0x{:04x}", val, x87_mantissa, exp, x87_exp);
  
        // Layout: [mantissa: 8 bytes][sign+exp: 2 bytes][padding: 6 bytes]
        result[0..8].copy_from_slice(&x87_mantissa.to_le_bytes());
  
        let sign_and_exp = ((sign as u16) << 15) | (x87_exp & 0x7FFF);
        result[8..10].copy_from_slice(&sign_and_exp.to_le_bytes());
  
        // Debug: print the result bytes
        println!("DEBUG result bytes: {:02x?}", &result[0..10]);
  
        result
    }
    fn x87_bytes_to_f64(bytes: &[u8; 16]) -> f64 {
        // Extract mantissa (8 bytes)
        let mantissa = u64::from_le_bytes([
            bytes[0], bytes[1], bytes[2], bytes[3],
            bytes[4], bytes[5], bytes[6], bytes[7],
        ]);
  
        // Extract sign and exponent (2 bytes)
        let sign_and_exp = u16::from_le_bytes([bytes[8], bytes[9]]);
        let sign = (sign_and_exp >> 15) & 1;
        let x87_exp = sign_and_exp & 0x7FFF;
  
        // Check for zero
        if mantissa == 0 && x87_exp == 0 {
            return if sign == 1 { -0.0 } else { 0.0 };
        }
  
        // Convert x87 to IEEE 754 double
        let ieee_exp = (x87_exp as i32) - 16383 + 1023;
  
        // Check for overflow/underflow
        if ieee_exp <= 0 {
            return if sign == 1 { -0.0 } else { 0.0 };
        }
        if ieee_exp >= 0x7FF {
            return if sign == 1 { f64::NEG_INFINITY } else { f64::INFINITY };
        }
  
        // Extract top 52 bits of mantissa (remove explicit leading 1)
        let ieee_mantissa = (mantissa << 1) >> 12;
  
        // Construct IEEE 754 double
        let ieee_bits = ((sign as u64) << 63) |
            ((ieee_exp as u64) << 52) |
            (ieee_mantissa & 0xFFFFFFFFFFFFF);
  
        f64::from_bits(ieee_bits)
    }
}
```

alot of debuggers have a UI you have a debugger ui in jetbrains IDEs and let me show the neovim UI here as well

and the UI captures and shows all the prints and logs to you, even if they were being written to std out since the process was being managed by the debugger we want a way to allow debuggers to capture that out put and show it to the user 

and so we want for any process that we launch through the debugger to intercept its out put and capture and process that in the debugger even if the process writes to the std output

so we will update the launch function to accept a parameter that will be the file descriptor of where we want to redirect the output to from the std out

and for now we will just pass it none but incase we want to build a UI for it we can update the api of our debugger to take the output file descriptor optionally as well

before we begin we will add two dependencies to our crate that we will use later on
```toml
libc = "0.2.177"
num-traits = "0.2.19"
```

in the attach.rs

```rust
Process::launch(&program_path, None)
```

and in the rdb_integ_test.rs

```rust
fn test_process_launch_success(){
    let proc = Process::launch("yes", None)
        .expect("Failed to launch process");
```
```rust
fn test_process_launch_nonexistent_process(){
    let proc = Process::launch("/random/non/existent/path/hopefully", None);
```

int the process.rs now lets update the launch function to accept an FD

```rust
    pub fn launch(
        program_path: &str,
        stdout_replacement: Option<RawFd>
    ) -> Result<Process, String>{}
```

we need to do this redirecting in the child process because thats where we exec into the debuggee 
```rust 
use nix::unistd::{close,dup2, execvp, fork, pipe, read, write, ForkResult, Pid};
use std::os::unix::io::{AsRawFd, FromRawFd, RawFd};
use std::fs::File;
const STDOUT_FILENO: RawFd = 1;


Ok(ForkResult::Child) => {
	// close(read_fd).ok(); // we only want to write from the child
	// we do this to intercept the output of the process that is being debugged
	// this will duplicate the stdout to what ever file descriptor the stdout_replacement is pointing to
	// maybe a file or a pipe
	if let Some(fd) = stdout_replacement {
		if libc::dup2(fd, libc::STDOUT_FILENO) < 0 {
			panic!("stdout_replacement failed");

		}
	}
	//let traceme_res = ptrace::traceme();
	//if let Err(e) = traceme_res {}
}
```

to make working with this debugger easier we want to be able to see what values are written into a particular register 
so when we read from it for our own benefit lets implement Display for RegisterValue so that we can simply print RegisterValue where ever we want
in process_registers.rs
```rust

impl fmt::Display for RegisterValue {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) //The lifetime `'_` means "infer the lifetime automatically"
    -> fmt::Result {
        match self {
            RegisterValue::U8(v) => write!(f, "0x{:02x} ({:3})", v, v),// {:02x} means we want the width of the formatted test to be 2 characters and if required pad with 0s and the x means lower case hexadecimal format, the width comes from the max possbile value of that data type 255 = 3 02x == ff == 255, the signed types will need one extra character for sign
            RegisterValue::U16(v) => write!(f, "0x{:04x} ({:5})", v, v),
            RegisterValue::U32(v) => write!(f, "0x{:08x} ({:10})", v, v),
            RegisterValue::U64(v) => write!(f, "0x{:016x} ({:20})", v, v),
            RegisterValue::I8(v) => write!(f, "0x{:02x} ({:4})", *v as u8, v),// we type cast it to un signed for aesthetic purposes, as we print the twos complement for the negative number which is cleaner
            RegisterValue::I16(v) => write!(f, "0x{:04x} ({:6})", *v as u16, v),
            RegisterValue::I32(v) => write!(f, "0x{:08x} ({:11})", *v as u32, v),
            RegisterValue::I64(v) => write!(f, "0x{:016x} ({:20})", *v as u64, v),
            RegisterValue::Float(v) => write!(f, "{:e} ({:.6})", v, v),// the e means scientific notation 1.5e2 == 1.5*10^2 nand .6 means 6 digits after decimal, because thats what a float can precisely store
            RegisterValue::Double(v) => write!(f, "{:e} ({:.15})", v, v),
            RegisterValue::LongDouble(v) => write!(f, "{:?}", v),// this question mark means it will display in the format implmente by the debug trait in the f128 crate
            RegisterValue::Byte64(bytes) => {
                write!(f, "[")?;
                for (i, byte) in bytes.iter().enumerate() {
                    if i > 0 {
                        write!(f, ",")?;
                    }
                    write!(f, "0x{:02x}", byte)?;
                }
                write!(f, "]")
            }
            // these two are just arrays of bytes byte 64 is of size 8 [u8; 8] and byte 128 is array of size 16
            // so we basically iterating and printing all the bytes in hexadecimal format
            // we reverse the iterator to display in big endian order most significant byte first
            RegisterValue::Byte128(bytes) => {
                write!(f, "[")?;
                for (i, byte) in bytes.iter().enumerate() {
                    if i > 0 {
                        write!(f, ",")?;
                    }
                    write!(f, "0x{:02x}", byte)?;
                }
                write!(f, "]")
            }
        }
    }
}
```

to do some manual testing lets add a few commands to our cli that will allow us to read and write values to registers
we have the command handler in process.rs
its the dispatch command function
![[{4D362CA2-679E-4EA7-90E4-F228AD0F730F}.png]]
![[{7B7E8EF6-588E-4268-9DCD-75560EB0DCE1}.png]]
![[{59E55317-FEBF-43E6-8E62-DF3CB5386419}.png]]
in the read_all_registers function make it return a Result
```rust
pub fn read_all_registers(&mut self) -> Result<&str, &str> {
// let regs_libc = ptrace::getregs(self.pid)
            //.unwrap_or_else(|e|panic!("Couldnt read GPR registers: {}", e));
            
	let regs_libc = ptrace::getregs(self.pid).map_err(|_| "Couldn't read GPR registers")?;
	
	//and 
	let result = unsafe {
		ptrace(
			PTRACE_GETFPREGS,
			self.pid.as_raw(),
			std::ptr::null_mut::<std::ffi::c_void>(),
			&mut fpregs as *mut _ as *mut std::ffi::c_void
		)
	};
	if result < 0 {
		return Err("Couldnt read to Floating point registers");
    }
    for {
	    if Errno::last() != Errno::UnknownErrno {
			return Err("Couldnt read Debug registers");
		}
    }
	Ok("Read all registers")
}
```
![[{A70B5D5C-8F09-4B7A-8948-3CD837C39A46}.png]]
make the write functions also return Result
==**make self mut in write_gprs, write_fprs and write_user functions**==
```rust
	fn write_fprs(&mut self) -> Result<&str, &str> {
        use nix::libc::{ptrace, PTRACE_SETFPREGS};
        let fprs = &self.proc_registers.data_.i387;
        let result = unsafe {
            ptrace(
                PTRACE_SETFPREGS,
                self.pid.as_raw(),
                std::ptr::null_mut::<std::ffi::c_void>(),
                fprs as *const _ as *const std::ffi::c_void
            )
        };
  
        if result < 0 {
            let errno = Errno::last();
            eprintln!("PTRACE_SETFPREGS failed: {:?}", errno);
            Err("Couldn't write to user")
        } else {
            Ok("Register Updated")
        }
    }

    fn write_gprs(&mut self) -> Result<&str, &str> {
        use nix::libc::{ptrace, PTRACE_SETREGS};
        let gprs = &self.proc_registers.data_.regs;
        let result = unsafe {
            ptrace(
                PTRACE_SETREGS,
                self.pid.as_raw(),
                std::ptr::null_mut::<std::ffi::c_void>(),
                gprs as *const _ as *const std::ffi::c_void
            )
        };
        if result < 0 {
            Err("Couldnt write to user")
        }else{
            Ok("Register Updated")
        }
    }
	fn write_user(&mut self,offset: usize, data: u64) -> Result<&str, &str> {
        println!("{}", data);
        use nix::libc::{ptrace, PTRACE_POKEUSER};
        let result = unsafe {
            ptrace(
                PTRACE_POKEUSER,
                self.pid.as_raw(),
                offset,
                data
            )
        };
        if result < 0 {
            Err("Couldnt write to user")
        }else{
            Ok("Register Updated")
        }
    }
```


![[{089EC4AC-D183-4C25-81BF-31C3B8A57898}.png]]
```rust
    pub unsafe fn write_to_user_by_register_id(
        &mut self,
        id: RegisterId,
        val: RegisterValue,
    ) -> Result<&str, &str> {
        let register = Register::by_id(id);
        let bytes = self.proc_registers.write_register(register, val);
        if register.register_type == RegisterType::Fpr {
            if bytes == std::ptr::null_mut() {
                return Err("Couldnt write to User");
            }
            println!("parsed value fprs: {}", val);
            self.write_fprs()
        } else {
            let offset = register.offset;
            let offset = offset & !0b111;
            if bytes == std::ptr::null_mut() {
                return Err("Couldnt write to User");
            }
            let bytes = RegisterValue::from_bytes::<u64>(bytes.add(offset));
            println!("----------------------------------------writing to user---------------------------------------------------------");
            self.write_user(offset, bytes)
        }
    }
```

![[{E915ABA5-D858-4A4D-8FCB-700577263ACD}.png]]
implement parse_value method in register_info and add str_to_f128, str_to_vector, str_to_int, str_to_float method outside the impl block
```rust
impl Regster {
	pub fn parse_value(&self,val_str: &str) -> Result<RegisterValue, &str> {
        match self.register_format {
            RegisterFormat::Uint => {
                match self.size {
                    1=>{
                        str_to_int::<u8>(val_str, 16) // base 16 is the standard for registers
                            .map(RegisterValue::U8)
                            .ok_or("Invalid format")
                    }
                    2=>{
                        str_to_int::<u16>(val_str, 16)
                            .map(RegisterValue::U16)
                            .ok_or("Invalid format")
                    }
                    4=>{
                        str_to_int::<u32>(val_str, 16)
                        .map(RegisterValue::U32)
                        .ok_or("Invalid format")
                    }
                    8=>{
                        str_to_int::<u64>(val_str, 16)
                            .map(RegisterValue::U64)
                            .ok_or("Invalid format")
                    }
                    _=>{Err("Invalid format")}
                }
            }
            RegisterFormat::DoubleFloat => {
                str_to_float::<f64>(val_str)
                    .map(RegisterValue::Double)
                    .ok_or("Invalid format")
            }
            RegisterFormat::LongDouble => {
                str_to_f128(val_str)
                    .map(RegisterValue::LongDouble)
                    .ok_or("Invalid format")
            }
            RegisterFormat::Vector => {
                match self.size {
                    8 => str_to_vector::<8>(val_str)
                        .map(RegisterValue::Byte64)
                        .ok_or("Invalid format"),
                    16 => str_to_vector::<16>(val_str)
                        .map(RegisterValue::Byte128)
                        .ok_or("Invalid format"),
                    _ => Err("Invalid format"),
                }
            }
        }
    }
}

fn str_to_f128(s: &str) -> Option<f128::f128> {
    let s =s.trim();
    match s.to_lowercase().as_str() {
        "nan" => return Some(f128::f128::NAN),
        "inf" | "infinity" => return Some(f128::f128::INFINITY),
        "-inf" | "-infinity" => return Some(f128::f128::NEG_INFINITY),
        _ => {}
    }
  
    let bytes = s.as_bytes();
    // Format: [sign][digits].[digits]
    let mut pos = 0;
    let len = bytes.len();
    if len == 0 {
        return None;
    }
    let is_negative = if bytes[pos] == b'-' {
        pos += 1;
        true
    } else if bytes[pos] == b'+' {
        pos += 1;
        false
    } else {
        false
    };
    if pos >= len {
        return None;
    }
    // Parse integer part
    let mut value = f128::f128::from(0);
    let mut has_digits = false;
  
    while pos < len && bytes[pos].is_ascii_digit() {
        let digit = (bytes[pos] - b'0') as i32;
        value = value * f128::f128::from(10) + f128::f128::from(digit);
        pos += 1;
        has_digits = true;
    }
    // Parse decimal part
    if pos < len && bytes[pos] == b'.' {
        pos += 1;
        let mut divisor = f128::f128::from(10);
  
        while pos < len && bytes[pos].is_ascii_digit() {
            let digit = (bytes[pos] - b'0') as i32;
            value = value + f128::f128::from(digit) / divisor;
            divisor = divisor * f128::f128::from(10);
            pos += 1;
            has_digits = true;
        }
    }
  
    if !has_digits { // "."
        return None
    }
  
    if pos != len {
        return None;
    }
  
    if is_negative {
        value = -value;
    }
  
    Some(value)
}
  
fn str_to_vector<const N: usize>(s: &str) -> Option<[u8; N]> {
    let mut bytes = [0u8; N];
    let mut chars = s.chars().peekable();
  
    if chars.next()? != '[' {
        return None;
    }
    for i in 0..N {
        // Parse hex byte (expecting format like "0x12")
        let hex_str: String = chars.by_ref().take(4).collect();
        bytes[i] = str_to_int::<u8>(&hex_str, 16)?;
        // Expect ',' or ']'
        match chars.peek()? {
            ',' if i < N - 1 => {
                chars.next();
            }
            ']' => {
                chars.next();
                break;
            }
            _ => return None,
        }
    }
    if chars.next().is_some() {
        return None;
    }
    Some(bytes)
}
  
fn str_to_int<T>(s: &str, base: u32) -> Option<T>
where
    T: num_traits::Num,
{
    let s = if base == 16 && s.starts_with("0x") {
        &s[2..]
    } else {
        s
    };
  
    T::from_str_radix(s, base).ok()
}
  
fn str_to_float<F: std::str::FromStr>(s: &str) -> Option<F> {
    s.parse::<F>().ok()
}
```
in process.rs simplify the wait_on_signal method
![[{A6BA3E28-3AB8-47CA-A1A4-1245BCFCE3BA}.png]]
update the main function 
![[{370A3778-DB2B-4E31-856C-BBA1126E8B0B}.png]]
and in the rustyline debug entry point add an unsafe block
![[{A6A736B7-4C7E-4089-9705-0AF9FA36F592}.png]]
## after we have the cli menu for commands test and show

```bash
cargo run yes
echo $((0xcafecafe))
> register write rsi 0xcafecafe
> register read rsi
> register write mm0 [0xba,0x5e,0xba,0x11]
> register read mm0
> register write xmm0 [0xba,0x5e,0xba,0x11]
> register write fsw 0xfff
```

write in hexa decimal format

The x87 FPU doesn't just let you write a value to a register like normal registers. You need to:

1. Put the actual value in `st0`
2. Tell the FPU "there's now 1 item on the stack" (via `fsw`)
3. Tell the FPU "st0 contains valid data" (via `ftw`)

this is needed if you want the assembly instructions to work properly, namely fstpt

- Is there actually data on the stack? (FSW tracks this)
- Is that data valid or garbage? (FTW tracks this)
The FPU stack is circular (st0 through st7). The CPU needs to know which of these 8 slots is currently the "Top".
- The **Status Word** is a 16-bit register that tracks the FPU state.
- **Bits 11-13** hold the "Top of Stack" pointer.
When you push a value onto the FPU stack, the pointer decrements (it goes down).
- Empty stack starts at pointer = `0`.
- If you push one item, the pointer wraps around to `7`.
Since we want to simulate a stack that has **one item** on it, we must manually set the "Top Pointer" bits to `7` (binary `111`). 0b0011100000000000 sets bits 11, 12, 13 to 1
Even if you put data in the register and point to it, the CPU needs to know if that data is valid garbage, or empty space.
- The **Tag Word** has 2 bits for every register (16 bits total).
- `0b11` means "Empty".
- `0b00` means "Valid Number".
We want the register at index 7 (our Top) to be **Valid**, and the other 7 registers to be **Empty**.
- We set the tag for index 7 to `00`.
- We set the tags for indices 0-6 to `11`. 0b0011111111111111 sets 00 for the first tag, 11 for all others