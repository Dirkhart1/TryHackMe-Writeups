# 1. crackme1
- A Simple task. Run the binanry
	- OS: Fedora/Linux , so to run the binary file we would use the command " ./crackme1 "
	- I received a error stating the file doesnt have permissions to run, to I I ran " chmod a+x crackme1 " to allow the file to be executed,
	- I received the flag "flag{not_that_kind_of_elf}" for running the file.

	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part1.png">


# 2. crackme2
- Finding the super secret password to obtain the flag
	- Running it normally with permissions from previous task would give "bash: ./crackme2: cannot execute: required file not found"
	- Ill keep this error in mind, but Ill focus on doing the first task "Find the super secret password"
		- I went to the website, dogbolt.org and went to its decomplier explorer to decompile the binary.
		- I found a IF statement that reveals the super secret password, which turns out to be "super_secret_password"
			- Submitted this to the first tasked and that was indeed the secret password.
	- Back to the main objective, which is also the second task for this part. Which is to find the flag.
		- The super password is likely a key compoment, so I'll make use of that once I figure out what to do.

	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part2-1.png">
	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part2-2.png">
	
	- This binary likely allows parameters to be included, this assumption is enforced further when analyizing the decomplied code.
		- I ran " ./crackme2 --parameter1 super_secret_password " and it failed.

	- The error from before says a file is required but isnt found, so I'll use that to direct my next move.
		- I ran "ldd crackme2" to see if I am able to even run it
			- returned not a dynamic executable.
     
		- After a short session of troubleshooting, it turns out I did not have the proper bit loader installed
			- Reason: Fedora may not have a 32x-bit loader installed by default, this file runs using that.
			- solution : run " sudo dnf install glibc.i686 " to install the loader.
     
	 <img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part2-3.png">
	 
	- Run ./crackme2 again and it worked
		- ran ./crackme2 super_secret_password and recieved the flag " flag{if_i_submit_this_flag_then_i_will_get_points} "

	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part2-4.png">

# 3. crackme3
-  Use basic reverse engineering skills to obtain the flag
	- The deescriptions is giving a hint to find out how to decompile the executable.
	- We will use dogbolt.org mentioned earlier to do that.
	- But first I want to see what would happened if I run it normally
		- returned "Usage : ./crackme3 PASSWORD ", telling me to find the password when decompling.
	- On the decomplier, the password is found
		- "ZjByX3kwdXJfNWVjMG5kX2xlNTVvbl91bmJhc2U2NF80bGxfN2gzXzdoMW5nNQ=="
	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part3-1.png">
	
	- run the executable agian with that as the password.
		- I got pranked and got denied.
	- I thought twice about the supposed password and quickly realized it is encoded, likely with base64 (strings ending with "==" is likely encoded with base64).
		- I went to base64deoce.org to decode it
		- Returned " f0r_y0ur_5ec0nd_le55on_unbase64_4ll_7h3_7h1ng5 "
	- The executable returned correct password, and the flag is the password used.

	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part3-2.png">

# 4. crackme4
- Analyze and find the password for the binary?
	- I ran the binary to see what is up.
		returned " Usage : ./crackme4 password
			   This time the string is hidden and we used strcmp "
	- I do not know what is "strcmp" so I looked up the term and saw it was a compare function that C uses.
		- It returned 0 if it is equal, +1 is the 1st value is greater than the 2nd, and -1 if the 2nd value is greater than the 1st.
	- I decomplied the code to see its structure.
		- There is two parts that would tell me what to do, which is to find what "local_28" is and how to read ""I]{I\x14V\x17{WAGQV\x17{TS@",0x13)"
	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part4-1.png">

	- I wanted a clue on my next move so I checkout out the hints and got " IDA / Radare2" so Ill be looking into and isntalling what tool this is.
		- Apparentky a reverse engineering framework and toolset.
	- I installed the radare2 from github and can use commands such as "r2 <><> " for later steps.
	- ran " r2 -d crackme4 " to enter debug mode
		- I ran "aaa" and "afl" to search for all the flags and functions used in this program
	- "sym.compare_pwd" is a interesting function, so I run "pdf @sym.compare_pwd" to see those lines.
	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part4-2.png">
	- From there, the "sym.imp.strcmp is the part we will look at.
		- It is comapring registers rdi and rdx, and is the register that carries the password.
	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part4-3.png">
	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part4-4.png">
	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part4-5.png">

	- We will place a breakpoint with the command "db 0x004006d2" so when I debug it, it will stop at that part for me to anaylize.
	- Run " ood <input_here> " and then run " dc".
	- You be stopped at the breakpoint, and you can see the rdi register using "px @rdi". And that the password is now visible for us to use.
	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part4-6.png">
	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part4-7.png">

# 5. crackme5
- What will be the input of the file to get output Good game ?

	- running the file shows that this one takes a input now. 
	- We will use the decomplier first to see the code structure before using radare2 again.
	- The decomplier only reveals that a strcmp function is being used again, so I will analyize using the radare2.
		- Ill run "r2 -AA -d crackme5 " so I can get the flags and enter debug immdeitally
		- I will then use " s @main " to see how the code is first ran.
	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part5-1.png">

	- I see that "sym.strcmp_" is a function so we will go to that with " s @sym.strcmp_"
	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part5-2.png">

	- Similar to the previous part, the password might be in the rdi and rax registers, so we will place a breakpoint with "db 0x00400764" and "ood string" and then run "dc" to run the program

	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part5-3.png">
	
	- We get the string "OfdlDSA|3tXb32~X3tX@sX`4tXtz" with it, I do not think this would be the flag but I will still try it out.
		- And it turns out that was the, as I got the response of "Good Game" from the program.
			- This really shows it never hurts to double check
   
	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part5-4.png">

# 6. Crackme6
- Analyze the binary for the easy password

	- Running the file returns " Good luck, read the source "
	- I can say with confidence I have to look into the source code again (Wait really!?).
	- Run "r2 -AA -d crackme6" to get started.
	- " s @main " so we can see the main function code.
	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part6-1.png">

	- There isn't much to look at this time, and so "  sym.compare_pwd " sticks out like a sore thumb.
	- Run "s @sym.compare_pwd" to see whats that about.
	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part6-2.png">

	- "sym.my_secure_test: looks veriy interesting, so we will check that out.
	- When you check out that section of the code, its very long, but there is comments after each compare "cmp" call.
		- When I read each one, I get "1337_pwd"
	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part6-3.png">
	
	- And when I run the program "./crackme6 1337_pwd" I get password OK, so that is the password.
		- Remember, Its always secure to leave a comment with a passwwork in it.
	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part6-4.png">

# 7. crackme7
- Analyze the binary to get the flag

	- Running the program shows that its a very simple program that says your name and number.
	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part7-1.png">

	- If I did know that this program has a secret, I wouldbe thought that this is just a normal first-time coder program.
	- But like we know by now we will have a hidden flag so we will enter it once again.

	- When we use r2 on it and s @main, we can see this code is pretty long.
	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part7-2.png">

	- Near the bottom  we see a out of reach condition to get "Wow such haxers" and it looks like we can reach it using "zi".
		- But if we do then we will be met with a Unknown Input error
	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part7-3.png">

	- However what if we tried to use the decimal conversion instead of ASCII?
		- We would get 31337.
	- If we place that number in the selection we would get the flag.
		- Why is it compared as a decimal and not ASCII?
		- I not a expert in assembly, but If I have to guess then I would assume that since it's not being compared in with a reigster and instead with a hex, then assembly does the check in decimal. Don't take my word for it though.
	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part7-4.png">



# 8. crackme8
- Analyze the binary and obtain the flag

	- Its a program that takes a password as a argument, so we will going into the source code again.
	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part8-1.png">

	- Its a seemlily short program, and we'll find a way to enter the access granted part.
	- We see in order to reach it we would need our input to be equal to a the hex "0xcafef00d"
	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part8-2.png">
	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part8-3.png">


	- in Decimal, that would be 3405705229, but if I use that as a argument it returned access denied. So thats cant not it this time.
	- The atoi call is interesting. What this is does is convert the ASCII to decimal by tyrning each ASCII value into a integer.
	- This is where things get wierd, so we get 2 different values if we use a  online converter.
		- One with a signed 2 complemnt, and a  average 10 decimal number.
	- Upon research I learned that if the most singificant bit in binary is a "1" then the value would be a negative, in which from the converter, thats the signed 2 complement.
		- That is "-889262067"
	<img src="https://github.com/Dirkhart1/TryHackMe-Writeups/blob/main/Reversing_ELF/Assets/part8-4.png">

	- if we use that in the program, we would get the flag.


# 9. Conclusion
- I wanted to get a glimpse into Reverse Engineering and relearn assembly so I went into this lab mostly blind. And I say mostly because I forgot some topics about assembly when I took a University class on it. I remember most of the structure of it, like registers, flags, and conditions. Luckily that's what this Room mostly includes. Radare2 is a tool I was not at all familiar with. So figuring out how to first run it took a while. And I got the gist of it now to do later work on Reverse Engineering. This lab will be y first step into Reverse Engineering and Assembly. I found it all to be interesting and enlightening.
