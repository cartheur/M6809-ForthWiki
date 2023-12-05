```
	.command -ai    ; output in Intel hex format
; ----------------------------------------------------------------------
; Simple test program for The Computer Journal's 6809 Uniprocessor
; B. Rodriguez  11 March 1993
; ----------------------------------------------------------------------
	.org h'ff00

	.equ duart,h'6000       ; base address of 2681 DUART
	.equ txrdy,4
	.equ rxrdy,1

; 2681 Initialization Table.  Each word in this table contains
; register-number:contents in the hi:lo bytes, respectively.
;
initbl: .dw h'022a ; Command Register A: reset rx, disable rx & tx
	.dw h'0230 ; Command Register A: reset tx
	.dw h'0240 ; Command Register A: reset error status
	.dw h'0210 ; Command Register A: reset MR pointer
	.dw h'0013 ; Mode Register A(1): 8 bits, no parity
	.dw h'0007 ; Mode Register A(2): 1 stop, RTS & CTS manual
	.dw h'01bb ; Clock Select A: tx & rx 9600 baud
	.dw h'0205 ; Command Register A: enable rx & tx
	.dw h'0a2a ; Command Register B: reset rx, disable rx & tx
	.dw h'0a30 ; Command Register B: reset tx
	.dw h'0a40 ; Command Register B: reset error status
	.dw h'0a10 ; Command Register B: reset MR pointer
	.dw h'0813 ; Mode Register B(1): 8 bits, no parity
	.dw h'0807 ; Mode Register B(2): 1 stop, RTS & CTS manual
	.dw h'09bb ; Clock Select B: tx & rx 9600 baud
	.dw h'0a05 ; Command Register B: enable rx & tx
	.dw h'0430 ; Aux Control Register: counter mode, xtal/16
	.dw h'062d ; Counter Upper, and
	.dw h'0700 ; Counter Lower: 2d00 hex = 50.000 msec
	.dw h'0d00 ; Output Port Configuration: all manual
	.dw h'0e03 ; Set Output Bits: OP0 and OP1 low
	.dw h'0500 ; Interrupt Mask Register: all disabled
endtbl:

; The test program enters here on a reset.
; This program doesn't use stacks, so the stack pointer doesn't
; need to be initialized.
;
entry:

; Initialize DUART from the table above.
;
	ldy #initbl
	ldx #duart
iloop:  ldd ,y++        ; fetch a:b from table
	stb a,x         ; store b at duart+a
	cmpy #endtbl
	bne iloop

; Simple memory test, to check locations 2000 to 3FFF hex.
;
outer:  ; Memory test, one pass.
	lda #h'2e       ; '.' character means good
	ldx #h'2000     ; starting address
mtest:  ldb #1          ; rotate this through all bit positions
bittest: stb ,x
	cmpb ,x
	bne bad
	aslb
	bne bittest
	bra good
bad:    anda #h'f5      ; error encountered: change '.' to '$'
good:   leax 1,x        ; next address,
	cmpx #h'4000    ;   and loop
	bne mtest

; Ouptut the character in the A register, over serial port A
;
tloop:  ldb duart+1
	andb #txrdy
	beq tloop
	sta duart+3

; Print all remaining ASCII characters up through 7F hex.
;
	inca
	bpl tloop

; Now wait for a received character, and echo it and all following
; characters.
;
rloop:  ldb duart+1
	andb #rxrdy
	beq rloop
	lda duart+3
	bra tloop

; The reset vector for the 6809 is located at address FFFE hex.
	.org h'fffe
	.dw  entry

	.end
```