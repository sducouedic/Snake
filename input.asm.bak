.equ HEAD_X,     0x1000 ; snake head's position on x-axis
.equ HEAD_Y,     0x1004 ; snake head's position on y-axis
.equ TAIL_X,     0x1008 ; snake tail's position on x-axis
.equ TAIL_Y,     0x100C ; snake tail's position on y-axis
.equ SCORE,      0x1010 ; score address
.equ GSA,        0x1014 ; game state array
.equ LEDS,       0x2000 ; LED addresses
.equ SEVEN_SEGS, 0x1198 ; 7-segment display addresses
.equ RANDOM_NUM, 0x2010 ; Random number generator address
.equ BUTTONS,    0x2030 ; Button addresses


; ----------------------MAIN---------------------------------------------
; 


terminate:
call restart_game
addi t0,zero,1
beq v0,t0,init
call draw_array
br terminate


init:
addi sp,zero,LEDS
call reset_GSA
stw zero, SCORE(zero)   	; On reinitialise edgecapture
stw zero, BUTTONS+4(zero)   	; On reinitialise edgecapture
addi v0,zero,0
addi a0,zero,0 ; x = 0			;| On definit la head a la coord(3,4) = GSA(28)
addi a1,zero,0 ; y = 0			;|
addi a2,zero,0 ; (0,0) == 0    	;|
stw a0, HEAD_X(zero)			;| On load les coordonnes de la head dans les addresses decrivant les coord de la head
stw a1, HEAD_Y(zero)			;|
stw a0, TAIL_X(zero)			;| On load ces memes coordonnees dans la tail car comme un element dans 
stw a1, TAIL_Y(zero)			;| le snake, tail == head
								;|---

addi a0,zero,4					;|--
stw a0,GSA(a2)					;| Head se deplace vers la droite, info ajoute dans GSA(a2) = GSA(28)
								;|--

addi a0,zero,0
addi s0,zero,0                  ; le score global des unites
addi s1,zero,0                  ; le score global des dizaines

call clear_leds	
call create_food	
call display_score
call draw_array
call wait
	
loop_main:

call get_input ;--- get user input
call hit_test ;--- see if there is a collision
;--- check what type of collision
addi t1,zero,1
addi t2,zero,2
beq v0,t2,terminate ;--- if type == 2 then terminate
addi a0,zero,0
;-- addi v0,zero,0
bne v0,t1,move ;--- if type == 1 pomme est mangee : then call createfood
addi a0,zero,1
call create_food
call draw_array
call increment_score
move:
call move_snake
call draw_array ;--- draw the array
call display_score

call wait
call restart_game ;--- check if we have to restart the game
addi t0,zero,1
beq v0,t0,init

br loop_main

;-------------------------------------------------------------------------

; BEGIN:clear_leds
clear_leds:
	; put LEDS[0_.2] to 0x00000000
	stw zero, LEDS(zero) ; Set LEDS[0] to 0
	stw zero, LEDS+4(zero) ; Set LEDS[1] to 0
	stw zero, LEDS+8(zero) ; Set LEDS[2] to 0
ret
; END:clear_leds

; BEGIN:increment_score
increment_score:

ldw t0, SCORE(zero)
addi t0,t0,1
stw t0, SCORE(zero)
ret
; END:increment_score

; BEGIN:reset_GSA
reset_GSA:
addi t0,zero,0
addi t1,zero,96

boucle_reset_GSA:
slli t2,t0,2
stw zero,GSA(t2)

addi t0,t0,1
blt t0,t1,boucle_reset_GSA
ret
; END:reset_GSA


; BEGIN:set_pixel
set_pixel:
; ao:x and a1:y
andi t0,a0,12 ; bits[3,2] de x
andi t1,a0,3 ; bits[1,0] de x
slli t2, t1,3 ;shift de 3 a gauche pour multiplier par 8 
add t3, a1,t2 ; t3 est la position du bit voulu a 1
addi t4,zero,1 ; vaut 1 que l'on va shifter de t3 positions
sll t4, t4, t3
ldw t5, LEDS(t0) ; On recupere la valeur initial du tableau de led
or t5,t5,t4 ; on ajoute le led
stw t5, LEDS(t0) ; Set LEDS[1] to 0
ret
; END:set_pixel


; BEGIN:draw_array
draw_array:

	;--- 2 access each cell of GSA and activate it if 1
;	nombrede32bits tmp;
;	for(i = 0; i < 3 ; i++){ (1)
;		 tmp = 0; (2)
;		i*=32; (3)
;		for(j = 0; j< 32; j++){ (4)
;			val k = if ( GSA[i+j] != 0) k == 0x80000000  else 0x00000000 (5)
;			tmp.shift >> 1 (6)
;			k or tmp (7)
;		}
;		LEDS[i] = tmp (8)
;	}	
															;|---
	addi t0,zero,0 ; t0 = tmp								;| on definit la variable tmp, qui represente la valeur de la LED numero i, et i a 0 pour la boucle for (1)
	addi t1,zero,0 ; t1 = i qu on initialise a zero			;|---
	addi t6,zero,32 ; t6 = 32
	addi t7,zero,1
	slli t7,t7,31 ; t7 = 0x80000000


	boucle: 											
	addi t0,zero,0 ; tmp = 0 (dans la boucle)				;|--- (2)
	add  t2,zero,t1; 										;|	on initialise tmp a 00...000 (32 fois) et i=*32 (3)
	slli t2,t2,5; t2 = i << 5 = i * 32						;| 	
	addi t3,zero,0 ; t3 = j qu on initialise a zero			;|---

	sous_boucle: 
	add t4,t2,t3; t4 = 32*i + j = addresse dans le GSA ; (4)
	slli t4,t4,2; to have a multiple of 4
	ldw t5,GSA(t4) ;t5 = valeur dans le tableau 

	addi t4,zero,0 ; t4 = k = 00...0  
	beq t5,zero,4 ; si t5 = 0 alors on laisse k = 0  ; (5)
	add t4,zero,t7

	srli t0,t0,1; tmp >> 1 pour pouvoir faire le masque (6)
	or t0,t0,t4 ; tmp = tmp or k pour le masque (7)
	addi t3,t3,1; j = j+i

	blt t3,t6,sous_boucle ; si j < 32, alors on reboucle	
	
	slli t4,t1,2
	stw t0,LEDS(t4) ; LEDS(i) = tmp final (8)
	addi t1,t1,1 ; i = i+1
	
	addi t4,zero,3
	blt t1,t4,boucle ; si i < 3, alors on reboucle sur la boucle principale

ret
; END:draw_array


; BEGIN:get_input
get_input:
	;--- 1 . load the value of the head
	ldw t0, HEAD_X(zero) 		; x coordinate of head
	ldw t1, HEAD_Y(zero) 		; y coordinate of head

	andi t2, t0, 12 	; bits[3,2] de x
	slli t2, t2, 3 		; shift de 5 a gauche pour multiplier par 32

	andi t3, t0, 3 		; bits[1,0] de x
	slli t3, t3,3 		; shift de 3 a gauche pour multiplier par 8

	add t4, t1,t3 		; t4 = head_y + 8 * head_x(1,0) = position dans une LED
	add t4, t4,t2 		; t4 = 32 * head_x(3,2) + 8 * head_x(1,0) + head_y = position dans GSA
	
	slli t4,t4, 2
	ldw t5, GSA(t4) 	; infos sur la head
	
	;--- 1.5 recuperer la direction actuelle de la tete
	addi t6,zero,7 ; masque sur les 3 premiers bits pour savoir la direction
	and t6,t6,t5 ; t6 est la direction actuelle de la tete
	

	;--- 2. get the Buttons info for the new direction
	ldw t0, BUTTONS+4(zero)		; on mets dans t0 la valeur de edgecapture 
	stw zero, BUTTONS+4(zero)   ; On reinitialise edgecapture

	;--- 3. get in which direction we want to go
	bne t0,zero,4	; s'il n'y a pas de changement de direction, on ne fait rien
	ret
	addi t7,zero,16
	bne t0,t7,4 ; si ce n'est pas le button reset
	ret
	addi t7,zero,0
	loop:
	addi t7,t7,1 ; count++
	addi t1,zero,1 ; Masque pour le bit 1
	and t2,t0,t1 ; on recupere la valeur du bit 1
	srli t0,t0,1 ; on shift de 1 vers la droite si on retourne dans la boucle
	bne t2,t1, loop ; 

	;---4. if directionActuelle + changement de direction = 5, alors c'est une direction oppose 1(left)+4(right) ou 2(up)+3(down)
	;registre utilise actuellement: t4(position de la head),t5(variable tete), t6 (direction actuelle),t7 (direction voulu)
	addi t0, zero ,5 ; direction oppose
	add t1,t6,t7
	
	;---5. update the head snake direction if not opposite direction !
	bne t0,t1,4
	ret ; ;if t1 == t0 (direction totale = 5) alors on ignore
	srli t5,t5,3
	slli t5,t5,3 ; on mets les 3 derniers bits a 0 et on additionne avec la direction voulu
	add t5,t5,t7
	stw t5, GSA(t4) ; on update la memoire
	ret
; END:get_input


; BEGIN:move_snake
move_snake:
	
	;--- 1 . load the value of the head
	ldw t0, HEAD_X(zero) 		; x coordinate of head
	ldw t1, HEAD_Y(zero) 		; y coordinate of head

	andi t2, t0, 12 ; x[3,2] << 2
	slli t2, t2, 3 	; x << 5 = 32*x

	andi t3, t0, 3 ; x[1,0]
	slli t3, t3,3 ; x << 3 = 8*x

	add t4, t1,t3 ; t4 = y + 8*x[1,0] = position dans une LED
	add t4, t4,t2 ; t4 = 32 * x[3,2] + 8 * x[1,0] + y = position dans GSA
	
	slli t4,t4, 2
	ldw t5, GSA(t4) 	; infos sur la head

	;--- 1.5 recuperer la direction actuelle de la tete
	addi t6,zero,7 ; masque sur les 3 premiers bits pour savoir la direction
	and t6,t6,t5 ; t6 est la direction actuelle de la tete
	
	;--- 2 . calcul de la nouvelle position(x,y) de la tete en fonction de sa direction
	addi t2,zero,1;	1 = left			;|---
	bne t6,t2,4; on saute si d != 1		;| si la direction == 1 (left), alors new_x = x - 1
	addi t0,t0, -1;	x = x - 1			;|---

	addi t2,zero,2; 2 = up  			;|---
	bne t6,t2,4; on saute si d != 2		;| si la direction == 2 (up), alors new_y = y - 1
	addi t1,t1,-1; y = y - 1			;|---

	addi t2,zero,3; 3 = down			;|---
	bne t6,t2,4; on saute si d != 3		;| si la direction == 3 (down), alors new_y = y + 1
	addi t1,t1,1; y = y + 1				;|---
	
	addi t2,zero,4; 4 = right			;|---
	bne t6,t2,4; on saute si d != 4		;| si la direction == 4 (right), alors new_x = x + 1
	addi t0,t0,1; x = x + 1				;|---

	;--- 3 . on stocke les nouvelles coordonnees de la tete dans HEAD_X/Y et GSA
	stw t0, HEAD_X(zero) ; t0 = head_x			
	stw t1, HEAD_Y(zero) ; t1 = head_y			
									

	andi t2, t0, 12 ; x[3,2] << 2		;|---
	slli t2, t2, 3 	; x << 5 = 32*x		;|
										;|
	andi t3, t0, 3 ; x[1,0]				;|	
	slli t3, t3,3 ; x << 3 = 8*x		;| t4 = position de la tete dans la GSA en fonction de new_x et de new_y
										;|	
	add t4, t1,t3 ; t4 = y + 8*x[1,0] = position dans une LED
	add t4, t4,t2 ; t4 = 32 * x[3,2] + 8 * x[1,0] + y = position dans GSA
										;| 
										;|---

	slli t4,t4, 2									;|---
	stw t6,GSA(t4); GSA(t4) = d			;| t6 contient toujours la direction de la tete (qui n'a pas change)
										;| et stocke cette direction dans la GSA a la nouvelle position
										;|---


	;--- 4. on recupere la position actuelle de la queue (tail) et sa direction
	ldw t0,TAIL_X(zero); t0 = tail_x
	ldw t1,TAIL_Y(zero); t1 = tail_y	

	andi t2, t0, 12 ; x[3,2] << 2		;|---
	slli t2, t2, 3 	; x << 5 = 32*x		;|
										;|
	andi t3, t0, 3 ; x[1,0]				;|	
	slli t3, t3,3 ; x << 3 = 8*x		;| t4 = position de la queue dans la GSA (en fonction de x et y)
										;|	
	add t4, t1,t3 ; t4 = y + 8*x[1,0] = position dans une LED
	add t4, t4,t2 ; t4 = 32 * x[3,2] + 8 * x[1,0] + y = position dans GSA
										;| 
										;|---

	;--- 4.5 recuperer la direction actuelle de la tail
	slli t4,t4, 2
	ldw t5, GSA(t4) ; infos de la tail	;|---
	addi t6,zero,7 ; masque sur [2,1,0]	;| t6 = direction actuelle de la queue
	and t6,t6,t5 ; 						;|---

	;--- 5 . calcul de la nouvelle position(x,y) de la queue en fonction de sa direction
	addi t2,zero,1;	1 = left			;|---
	bne t6,t2,4; on saute si d != 1		;| si la direction == 1 (left), alors new_x = x - 1
	addi t0,t0, -1;	x = x - 1			;|---

	addi t2,zero,2; 2 = up				;|---
	bne t6,t2,4; on saute si d != 2		;| si la direction == 2 (up), alors new_y = y - 1
	addi t1,t1,-1; y = y - 1			;|---

	addi t2,zero,3; 3 = down			;|---
	bne t6,t2,4; on saute si d != 3		;| si la direction == 3 (down), alors new_y = y + 1
	addi t1,t1,1; y = y + 1				;|---
	
	addi t2,zero,4; 4 = right			;|---
	bne t6,t2,4; on saute si d != 4		;| si la direction == 4 (right), alors new_x = x + 1
	addi t0,t0,1; x = x + 1				;|---
	
	;--- 6 . on stocke les nouvelles coordonnees de la queue dans TAIL_X/Y et on supprime l'ancienne queue de la GSA
	beq a0,zero,4		; Ne supprime pas la queue s'il y a une collision avec la nourriture
	ret
	stw t0, TAIL_X(zero) ; t0 = tail_x			
	stw t1, TAIL_Y(zero) ; t1 = tail_y		
	stw zero,GSA(t4)	; GSA(ancienne position de la TAIL) = 0 ce qui veut dire ne contient rien

ret
; END:move_snake


; BEGIN:create_food
create_food:

	;--- 1. on genere l'indice aleatoire
	genere:
	ldw t0, RANDOM_NUM(zero)
;	addi t0,t0,35
	andi t0,t0,255 ; on retient seulement les huit premiers bits du random

	;--- 2. teste sur l'indice
	;--- 2.1 teste sur le bounds
	addi t1,zero,96
	bge t0,t1, genere
	blt t0,zero, genere

	;--- 2.2 teste si ca collide
	slli t0,t0,2
	ldw t1, GSA(t0)
	bne t1,zero,genere

	;--- 3. si ca ne collide pas, on cree le food
	addi t2,zero,5 ; on mets la valeur 5 representant la pomme dans le GSA
	stw t2, GSA(t0)
	
ret
; END:create_food


; BEGIN:hit_test
hit_test:

;--- 1 trouver la next position du snake's head based on the diretion vector of the snake's head
	;--- 1 . load the value of the current head
	ldw t0, HEAD_X(zero) 		; x coordinate of head
	ldw t1, HEAD_Y(zero) 		; y coordinate of head

	andi t2, t0, 12 ; x[3,2] << 2
	slli t2, t2, 3 	; x << 5 = 32*x

	andi t3, t0, 3 ; x[1,0]
	slli t3, t3,3 ; x << 3 = 8*x

	add t4, t1,t3 ; t4 = y + 8*x[1,0] = position dans une LED
	add t4, t4,t2 ; t4 = 32 * x[3,2] + 8 * x[1,0] + y = position dans GSA
	
	slli t4,t4, 2
	ldw t5, GSA(t4) 	; infos sur la head

	;--- 1.5 recuperer la direction actuelle de la tete
	addi t6,zero,7 ; masque sur les 3 premiers bits pour savoir la direction
	and t6,t6,t5 ; t6 est la direction actuelle de la tete
	

	;--- 2 . calcul de la nouvelle position(x,y) de la tete en fonction de sa direction
	addi t2,zero,1;	1 = left			;|---
	bne t6,t2,4; on saute si d != 1		;| si la direction == 1 (left), alors new_x = x - 1
	addi t0,t0, -1;	x = x - 1			;|---

	addi t2,zero,2; 2 = right			;|---
	bne t6,t2,4; on saute si d != 2		;| si la direction == 2 (up), alors new_y = y - 1
	addi t1,t1,-1; y = y - 1			;|---

	addi t2,zero,3; 3 = down			;|---
	bne t6,t2,4; on saute si d != 3		;| si la direction == 3 (down), alors new_y = y + 1
	addi t1,t1,1; y = y + 1				;|---
	
	addi t2,zero,4; 4 = right			;|---
	bne t6,t2,4; on saute si d != 4		;| si la direction == 4 (right), alors new_x = x + 1
	addi t0,t0,1; x = x + 1				;|---

	;--- 3. on teste les boundaries
	addi t3,zero,12
	addi t4,zero,8
	blt t0,zero,collision
	blt t1,zero,collision
	bge t0,t3,collision
	bge t1,t4,collision
	
	;--- 4. on load la valeur de la GSA (x,y) est dans le domaine
	;--- 4.1 . on calcule ces coordonnes (x,y) en absolu dans la GSA
	andi t2, t0, 12 ; x[3,2] << 2		;|---
	slli t2, t2, 3 	; x << 5 = 32*x		;|
										;|
	andi t3, t0, 3 ; x[1,0]				;|	
	slli t3, t3,3 ; x << 3 = 8*x		;| t4 = position de la tete dans la GSA en fonction de new_x et de new_y
										;|	
	add t4, t1,t3 ; t4 = y + 8*x[1,0] = position dans une LED
	add t4, t4,t2 ; t4 = 32 * x[3,2] + 8 * x[1,0] + y = position dans GSA
										;| 
										;|---

	;--- 4.2 on load la valeur de la GSA
	slli t4,t4, 2	
	ldw t6,GSA(t4)

	;--- 5. on teste le body et la pomme
	addi t3,zero,5
	beq t6,t3,point
	bne t6,zero,collision
	br nothing
	
	collision:
	addi v0,zero,2
	ret

	point:
	addi v0,zero,1
	ret
	
	nothing:
	addi v0,zero,0
	ret

ret
; END:hit_test


; BEGIN:display_score
display_score:

;t0 est le score des unites
;t1 est le score des dizaines
;il suffit de bien les afficher


;--- read from memory
ldw t0, SCORE(zero)
addi t1,zero,0
addi t3,zero,10
;--- get unit score
get_dizaine:
blt t0,t3,get_unit
addi t1,t1,1
addi t0,t0,-10
;--- get dizaine score
get_unit:

slli t0,t0,2
slli t1,t1,2

;--- print
ldw t0,font_data(t0)
ldw t1,font_data(t1)
ldw t2,font_data(zero)


stw t0, SEVEN_SEGS+12(zero) ; car seven_segs [0,1,2,3]
stw t1, SEVEN_SEGS+8(zero) ; car seven_segs [0,1,2,3]
stw t2, SEVEN_SEGS+4(zero) ; car seven_segs [0,1,2,3]
stw t2, SEVEN_SEGS(zero) ; car seven_segs [0,1,2,3]

ret
; END:display_score


; BEGIN:restart_game
restart_game:
	;--- 2. get the Buttons info for the restart
	ldw t0, BUTTONS+4(zero)		; on mets dans t0 la valeur de edgecapture 

	;---
	addi t1,zero,16
	addi v0,zero,0
	bge t0,t1,4
	ret
	addi v0,zero,1
	
ret
; END:restart_game

; BEGIN:wait
wait:
addi t0,zero,5000
slli t0,t0,10
wait_boucle:
addi t0,t0,-1
bne t0,zero,wait_boucle
ret
; END:wait


font_data:
	.word 0xFC ; 0
	.word 0x60 ; 1
	.word 0xDA ; 2
	.word 0xF2 ; 3
	.word 0x66 ; 4
	.word 0xB6 ; 5
	.word 0xBE ; 6
	.word 0xE0 ; 7
	.word 0xFE ; 8
	.word 0xF6 ; 9

