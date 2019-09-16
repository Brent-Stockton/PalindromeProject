# PalindromeProject
PLP Tool 5.2 MIPS Assembly Language Palindrome Checker

# Brent Stockton
# CSE 230
# Porject 3


.org 0x10000000

# Initialize the Stack
li $sp, 0x10fffffc
# Initialize UART Command Register "Write-Only"
li $s1, 0xf0000000
# Initialize UART Status Register "Read-Only"
li $s2, 0xf0000004
# Initialize UART Recieve Buffer "Read-Only"
li $s3, 0xf0000008

# Initialize any registers you will be using here.
# It can be helpful to include a comment about a register's purpose
# next to an initialization at the start of the program for reference.

j main
nop

# Label pointing to 100 word Array
arrayPointer:			
	.space 100

readInput: 
	#bit mask for 2nd bit
	li $s5, 0b10 

	# Wait until a status register determines that a character is waiting
	push $ra
	jal waitInput
	nop
	
	# look for a "."
	move $s6, $zero 
	push $s5
	jal characterReader
	nop
	pop $s5
	pop $ra

	# Clear 
	sw $s5, 0($s1)

	# If "." Go back to the return address
	bne $s6, $zero, gotoReturn
	nop
	
	# Go back to start of readInput
	j readInput
	nop

# Period Reader
periodReader:
	addiu $s6, $s6, 1
	jr $ra
	nop


# standby until status register = 2 
waitInput:
	lw $t1, 0($s2)
	and $t0, $t1, $s5
	bne $t0, $zero, gotoReturn
	nop
	j waitInput
	nop

#Checks to see if it is a letter that is uppercase or lowercase. If it is lower case it converts to uppercase by subracting by 32.
characterReader:
	lw $s5, 0($s3) 
	li $t0, 46 
	beq $t0, $s5, periodReader
	addiu $t0, $t0, 19 
	slt $t1, $s5, $t0
	bne $t1, $zero, gotoReturn
	addiu $t0, $t0, 57 
	slt $t1, $t0, $s5 
	bne $t1, $zero, gotoReturn
	addiu $t0, $t0, -31 
	slt $t1, $s5, $t0  
	bne $t1, $zero, charactertoStack
	addiu $t0, $t0, 6 
	slt $t1, $s5, $t0 
	bne $t1, $zero, gotoReturn
	addiu $s5, $s5, -32 
	j charactertoStack
	nop

# Puts character into Array/Stack
charactertoStack:
	sw $s5, 0($s0)
	addiu $s0, $s0, 4
	jr $ra
	nop


#Checks to see if it is a Palindrome
checkforPalindrome:
	addiu $s0, $s0, -4
	beq $s0, $s4, isPalindrome
	nop
	lw $t0, 0($s0) 
	lw $t1, 0($s4) 
	bne $t0, $t1, gotoReturn
	addiu $s4, $s4, 4
	beq $s0, $s4, isPalindrome
	nop
	j checkforPalindrome
	nop
	
#Correct palindrome and is assigned 1 for "Yes"
isPalindrome:
	addiu $a0, $a0, 1
	jr $ra

#Back to Return Address
gotoReturn:
	jr $ra
	nop

# Main program
main:
	# TODO: write your primary program within this loop
	li $s0, arrayPointer 
	li $s4, arrayPointer 

	jal readInput
	nop
	
	move $a0, $zero 
	jal checkforPalindrome
	nop
	call project3_print	

	j main
	nop






