;************************************************************************
;* Periferals addreses
;************************************************************************
P_OUT1    EQU 0A000H  ; displays input address
P_OUT2    EQU 0C000H  ; keyboard line input address
PIN       EQU 0E000H  ; keyboard columns output address

;************************************************************************
;* General purpose mask 
;************************************************************************
MASK     EQU 000FH
MASK2    EQU 00F0H  

;************************************************************************
;* MediaCenter commands and andresses
;************************************************************************
SEL_SCREEN        EQU 6004H
LINE              EQU 600AH
COLUMN            EQU 600CH
PIXEL             EQU 6012H
REMOVE_WARNING    EQU 6040H
SEL_BACKGROUND    EQU 6042H
SEL_SOUND_VOL     EQU 604AH 
PLAY_SOUND        EQU 605AH
FRONT_MEDIA       EQU 6046H
DEL_FRONT_MEDIA   EQU 6044H

;**************************************************************************
;* Colors and useful constants
;**************************************************************************
NR_OBJECTS    EQU 4       ; nr of objects
BLUE          EQU 0F0FFH  ; red color
GREEN         EQU 0F0F0H  ; green color
CREAM         EQU 0FFFEH  ; cream color
BLACK         EQU 0F000H  ; black color
GRAY          EQU 0FCCCH  ; gray color
PURPLE        EQU 0FF0FH  ; purple color
BLANK         EQU 00000H  ; erase color
DELAY         EQU 0400H   ; delay constant
MAX_COLUMN    EQU 64      ; nr of columns
MAX_LINE      EQU 32      ; nr of lines
RANGE         EQU 9       ; missile range

;**************************************************************************
;* Custom constants and macros
;**************************************************************************
; used as mediacenter screen selector
SCREEN_0     EQU 0
SCREEN_1     EQU 1

; used as arguments for update_display routine
ADD_OP            EQU 0
SUBTRACT_OP       EQU 1
GOOD_METEOR_VALUE EQU 10
DECAY_VALUE       EQU 5
SHOOT_VALUE       EQU 5
HIT_ROVER_VALUE   EQU 5

; general purpose
MAX_ENERGY       EQU 100
ROVER_INIT_X     EQU 32
ROVER_INIT_Y     EQU 28

; images
PAUSE_IMAGE         EQU 1
START_IMAGE         EQU 2
END_IMAGE           EQU 3
ROVER_EXPLODE_IMAGE EQU 4

; sounds
FIRE_SOUND        EQU 0
EXPLOSION_SOUND   EQU 1
GOOD_METEOR_SOUND EQU 2
;**************************************************************************
;* Processes stacks, LOCKS and Control variables
;**************************************************************************
PLACE    1000H
STACK    0100H
SP_main:               ; main process stack 

STACK    0100H
SP_keyboard:           ; keyboard process stack ( process that handles keyboard input )

STACK    0100H
SP_objects_update:     ; objects_update process stack ( process that manages meteors and enemy rovers)

STACK    0100H
SP_rover:              ; rover process stack  ( process that manages rover's movement )

STACK    0100H
SP_energy:             ; energy process stack ( process that handles rover's energy )

STACK    0100H
SP_fire:               ; fire process stack  ( process that manages the missile )

STACK   0100H
SP_start:              ; start process stack ( process controls the start and restart of the game )

IS_PAUSED:         ; identifies global pause state, 1 -> paused, 0 -> unpaused
	WORD 1

END:               ; identifies global end state, 1 -> game is over, 0 -> game still going
	WORD 0

FIRE:              ; identifies fire state, 1 -> fire happening, 0 -> fire not happening
	WORD 0

ENERGY_COUNTER:    ; current rover energy
	WORD 100

KB_KEY:            ; LOCK for main rotine, stores information on valid clicked keys, LOCK written by keyboard process
	LOCK 0

CONT_KB_KEY:       ; LOCK for rover process, stores information on valid keys holded, LOCK written by keyboard process
	LOCK 0

; interruptions LOCKs
INT_0:
	LOCK 0

INT_1:
	LOCK 0

INT_2:
	LOCK 0

; interruptions table with interruption's labels
interruptions:    
	WORD int0
	WORD int1
	WORD int2

;**************************************************************************
;* Data structures
;**************************************************************************
; Spacerover descriptor
SPACEROVER:
	WORD 4, 5                               ; height and width
	WORD 0,     0,     CREAM, 0,     0      ; first line
	WORD CREAM, 0,     CREAM, 0,     CREAM  ; second line
	WORD CREAM, CREAM, CREAM, CREAM, CREAM  ; third line
	WORD 0,     CREAM, 0,     CREAM, 0      ; fourth line

; Spacerover coordinates
SPACEROVER_COORDS:
	WORD ROVER_INIT_X  ; X coordinate
	WORD ROVER_INIT_Y  ; Y coordinate

; NOTE: all objects descriptors and coordinates in this section will follow the same structure as above

MISSILE:
	WORD 1, 1
	WORD PURPLE
	WORD PURPLE

MISSILE_COORDS:
	WORD 0
	WORD 0

; Object descriptors
; 1x1
OBJECT_1:
	WORD 1, 1
	WORD GRAY 
	
; 2x2
OBJECT_2:
	WORD 2, 2
	WORD GRAY, GRAY
	WORD GRAY, GRAY

; 3x3
ENEMY_ROVER_3:
	WORD 3, 3
	WORD BLACK, 0,     BLACK
	WORD 0,     BLACK, 0
	WORD BLACK, 0,     BLACK

; 4x4
ENEMY_ROVER_4:
	WORD 4, 4
	WORD BLACK, 0,     0,     BLACK
	WORD 0,     BLACK, BLACK, 0
	WORD BLACK, 0,     0,     BLACK
	WORD BLACK, 0,     0,     BLACK
	
; 5x5
ENEMY_ROVER_5:
	WORD 5, 5
	WORD BLACK, 0,     BLACK, 0,     BLACK
	WORD BLACK, 0,     BLACK, 0,     BLACK
	WORD 0,     BLACK, BLACK, BLACK, 0
	WORD BLACK, 0,     BLACK, 0,     BLACK
	WORD BLACK, 0,     BLACK, 0,     BLACK

; Meteors descriptors
; 3z3 
METEOR_3:
	WORD 3, 3
	WORD 0,     GREEN, 0
	WORD GREEN, GREEN, GREEN
	WORD 0,     GREEN, 0

; 4x4
METEOR_4:
	WORD 4, 4
	WORD 0,     GREEN, GREEN, 0
	WORD GREEN, GREEN, GREEN, GREEN
	WORD GREEN, GREEN, GREEN, GREEN
	WORD 0,     GREEN, GREEN, 0

; 5x5
METEOR_5:
	WORD 5, 5
	WORD 0,     GREEN, GREEN, GREEN, 0
	WORD GREEN, GREEN, 0,     GREEN, GREEN
	WORD GREEN, 0,     0,     0,     GREEN
	WORD GREEN, GREEN, 0,     GREEN, GREEN
	WORD 0,     GREEN, GREEN, GREEN, 0

; Meteors and enemy spacerovers information. X coordinate, Y coordinate, type identifier (0 - enemy ship, 1 - meteor)
OBJECTS_COORDS:
	WORD 32, 0, 0  ; 1st obejct
	WORD 32, 0, 0  ; 2nd object
	WORD 32, 0, 0  ; 3rd object
	WORD 32, 0, 0  ; 4th object

EXPLOSION:
	WORD 5, 5
	WORD 0,    BLUE, 0,    BLUE, 0
	WORD BLUE, 0,    BLUE, 0,    BLUE
	WORD 0,    BLUE, 0,    BLUE, 0
	WORD BLUE, 0,    BLUE, 0,    BLUE
	WORD 0,    BLUE, 0,    BLUE, 0

; Used to erase explosion of object when it happens, if X = 0 then no explosion happened
EXPLOSION_COORDS:
	WORD 0, 0

; Table with addresses of all forms of a meteor
METEOR_FORMS:
	TABLE 060H
; Table with addresses of all forms of a spacerover
ENEMY_ROVER_FORMS:
	TABLE 060H

;******************************************************************************
;* Main game control
;******************************************************************************
PLACE 0000H
	MOV   SP, SP_main  ; init stack
	MOV   BTE, interruptions
	CALL  setup        ; setup program (media center and objects)
	EI0
	EI1
	EI2
	EI

main:
; processes
	CALL  keyboard        ; keyboard process
	CALL  objects_update  ; updates meteors
	CALL  rover           ; moves ship 
	CALL  energy          ; updates energy ( hex display )
	CALL  fire            ; handles missile firing
	CALL  start           ; handles start and restart of game

; main loop of process 
loop_main:
	MOV   R0, [KB_KEY]   ; LOCK processes untill keyboard key is clicked
	CMP   R0, 0          ; R0 = 0 -> no valid click
	JZ    loop_main

; handle control keys -> pause game 
	MOV   R1, 0010b
	AND   R1, R0                  ; checks if the key is on the 2nd line
	JNZ   fire_key                ; if 2nd line key was clicked, it was the fire key (move is written to a different LOCK)
	MOV   R1, 1000b
	AND   R1, R0                  ; checks if the key is on the 4th line	      
	JZ    loop_main
	SHR   R0, 4                   ; get column info, 4-7 bits on 0-3 bits
	MOV   R1, 0010b
	AND   R1, R0                  ; checks if the key is the D key (pauses/unpauses game)
	JNZ   d_key               
	MOV   R1, 0100b
	AND   R1, R0                  ; checks if the key is the E key (ends game)
	JZ    loop_main
; set game to end state
	MOV   R2, [END]               ; END flag
	CMP   R2, 1                   ; if game is in end state, don't end it again
	JZ    loop_main
	MOV   R2, 1		      
	MOV   [IS_PAUSED], R2         ; pauses the game
	MOV   [END], R2               ; puts the game in end state
	MOV   R2, END_IMAGE
	MOV   [FRONT_MEDIA], R2       ; displays the pause message
	JMP   wait_key

unpause:
	MOV   R2, 0
	MOV   [IS_PAUSED], R2         ; reset PAUSE flag
	MOV   [DEL_FRONT_MEDIA], R2   ; remove the pause message
	JMP   wait_key

d_key:  
	MOV   R2, [END]               ; END flag
	CMP   R2, 1                   ; if game is in end state, don't unpause processes
	JZ    loop_main
	MOV   R0, [IS_PAUSED]
	CMP   R0, 1                    ; if the game is paused, unpause it
	JZ    unpause
	MOV   R2, 1		      
	MOV   [IS_PAUSED], R2         ; else, pauses the game
	MOV   [FRONT_MEDIA], R2       ; activates the pause message
wait_key:
	YIELD
	MOV   R2, [END]               
	CMP   R2, 1                    ; if END flag is 1, hold proccess
	JZ    wait_key     
	JMP   loop_main

; fire process
fire_key:
	SHR  R0, 4                     ; ignore line information bits 0-3, 0-3 bits now store column info
	AND  R0, R1                    ; check for click on fire key, 5 (R1 has 0010b at this point)
	JZ   loop_main
	MOV  R1, [IS_PAUSED]
	CMP  R1, 1                     ; if game is paused, do nothing
	JZ   loop_main
	MOV  R1, 1
	MOV  R0, [FIRE]                ; FIRE flag, if 0 -> no missile fired, if 1 -> missile mid air
	CMP  R0, R1                    ; if missile is mid air don't do anything
	JZ   loop_main

	MOV  [FIRE], R1                   ; activate FIRE flag
	MOV  R2, MISSILE_COORDS           ; missile coordinates address
	MOV  R1, [SPACEROVER_COORDS]      ; R1 <- rover's X coord
	ADD  R1, 2                        ; R1 + 2 will give us X of the middle of the rover
	MOV  [R2], R1                     ; update missile X coordinate
	MOV  R1, [SPACEROVER_COORDS + 2]  ; R1 <- rover's Y coord
	DEC  R1                           ; R1 - 1 will give us Y of the pixels on top of the rover
	MOV  [R2 + 2], R1                 ; update missile Y coordinate
	MOV  R1, MISSILE                  ; missile object descriptor
	MOV  R0, 1                        ; missile screen
	CALL draw_object
	MOV  R9, SHOOT_VALUE             ; argument of value of energy decay per missile fired for update_display
	MOV  R8, SUBTRACT_OP             ; argument of operation for update_display
	CALL update_display
	MOV  R0, FIRE_SOUND
	MOV  [PLAY_SOUND], R0            ; play firing sound
	JMP  loop_main

;******************************************************************************
;*
;*                           FIRE PROCESS 
;*
;******************************************************************************
PROCESS SP_fire
fire:
	MOV   R0, [INT_1]        ; LOCK process
	MOV   R1, [IS_PAUSED]    
	CMP   R1, 1              ; if IS_PAUSED flag is 0, game is running, if 1, game is paused
	JZ    fire
	MOV   R1, [FIRE]         ; FIRE flag, if 0 -> no missile fired, if 1 -> missile mid air
	CMP   R1, 0              ; if there is not missile being fired, don't do anything
	JZ    fire

	MOV   R0, SCREEN_1        ; mediacenter screen display argument
	MOV   R1, MISSILE         ; missile object descriptor argument
	MOV   R2, MISSILE_COORDS  ; missile object coordinates argument
	CALL  erase_object        ; erase current missile 
	MOV   R3, [R2 + 2]        ; current missile Y coordinate
	DEC   R3                  ; decrement Y, going up 1 pixel
	MOV   [R2 + 2], R3        ; update missile Y coordinate
	CALL  draw_object         ; draw missile in new coordinates
	MOV   R4, RANGE           ; missile maximum range
	MOV   R5, MAX_LINE        ; max line
	SUB   R5, 4               ; decrement rover's height
	SUB   R5, R4              ; line of missile's maximum range
	CMP   R5, R3              ; if current Y coordinate = max range -> missile ends
	JNZ   fire

	CALL  erase_object       ; erase missile from its last position
	MOV   R0, 0
	MOV   [R2], R0           ; reset missile X coordinate to 0
	MOV   [R2 + 2], R0       ; reset missile Y coordinate to 0
	MOV   [FIRE], R0         ; reset FIRE flag to 0

	JMP fire

;******************************************************************************
;*
;*                           ROVER PROCESS 
;*
;******************************************************************************
PROCESS SP_rover
rover:
	MOV   R0, [CONT_KB_KEY]    ; LOCK process 
	MOV   R1, [IS_PAUSED]      ; check global status variable
	CMP   R1, 1                ; if 1, game is paused
	JZ    rover
	MOV   R1, 0010b
	AND   R1, R0           ; if 0b0010 AND line nr = 0 -> second line was not clicked, no action
	JZ    rover
	SHR   R0, 4
	MOV   R1, 0101b
	AND   R0, R1           ; if column number AND 0b0101 = 0 -> no movement keys clicked, no action 
	JZ    rover
move:
; determine rover movement and detect collision
	MOV   R2, SPACEROVER_COORDS
	MOV   R1, [R2]            ; R1 will calculate the rover collision pixel in the X axis
	CMP   R0, 1               ; check movement direction, 1 -> left, else right
	JZ    left 
	MOV   R4, 1               ; increment X coordinate value
	INC   R1                  ; increment collision pixel 
	MOV   R2, MAX_COLUMN      ; right border X coord 
	ADD   R1, 5               ; add rover width to collision pixel
	CMP   R2, R1              ; check if collision pixel X coord is greater or equal to right side border
	JMP   detect_collision
left:
	MOV   R4, -1              ; increment X coordinate value
	DEC   R1                  ; decrement collision pixel
	MOV   R3, 0               ; left border X coord
	CMP   R1, R3              ; check if collision pixel X coord is less or equal to left side border
detect_collision:
	JN    rover_ret           ; if collision, no movement
	MOV   R1, DELAY           ; delay movement
rover_delay:
	
	YIELD

	DEC   R1
	JNZ   rover_delay
; update rover
	MOV   R0, 0                  ; mediacenter screen argument
	MOV   R1, SPACEROVER         ; spacerover descriptor argument
	MOV   R2, SPACEROVER_COORDS  ; spacerover coordinates argument
	CALL  erase_object           ; erase current rover 
	MOV   R3, [R2]               ; R3 -> current rover X coordinate
	ADD   R3, R4                 ; calculate new rover X coordinate 
	MOV   [R2], R3               ; update rover X coordinates
	CALL  draw_object            ; draw rover in new coordinates
rover_ret:
	JMP rover


;*****************************************************************************************;
;
;                                  ENERGY PROCESS
;
;*****************************************************************************************;
PROCESS SP_energy
energy:
	MOV  R0, [INT_2]          ; LOCK process
	MOV  R1, [IS_PAUSED]      ; check if game is paused 
	CMP  R1, 1                ; if 1, game is paused
	JZ   energy
	MOV  R1, [END]            ; check if game has ended
	CMP  R1, 1                ; if 1, game has ended
	JZ   energy
	MOV  R9, DECAY_VALUE      ; argument value of energy decay per cLOCK tick for update_display
	MOV  R8, SUBTRACT_OP      ; argument of operation for update_display
	CALL update_display
	JMP  energy

;********************************************************************************
; Recieves a value and an operation identifier and adds or subtracts it from the current energy counter
; Arguments:
; R9 - Value to be added/subtracted
; R8 - Operation, 0 add, 1 subtract
;********************************************************************************
update_display:
	PUSH R1
	MOV  R1, [ENERGY_COUNTER]       ; current energy value
	CMP  R8, 0                      ; identify operation, if 0 -> ADD, if 1 -> SUB
	JNZ  subt
	ADD  R1, R9                     ; ADD operation
	JMP  upd
subt:
	SUB  R1, R9                     ; SUB operation
upd:
	MOV  R9, R1                      ; convert_decimal argument
	JP   write_to_dis                ; if display is 0 or negative, R9 <- 0 
	MOV  R9, 0
	MOV  R1, 0
write_to_dis:
	MOV  [ENERGY_COUNTER], R1        ; update energy variable
	CALL convert_decimal             ; convert to decimal and write to display
end_update:
	POP  R1
	RET

;********************************************************************************
;* Converts hexadecimal number to a decimal representation in hexadecimal
;* Arguments:
;* R9 - Hexadecimal value to be displayed as decimal
;********************************************************************************
; by adding 6 to an hexadecimal number for each time 10 can be subtracted from it, we can represent [0, 100[ hexadecimal numbers in their decimal notation
; in the end, to include 100, the same process is applied for the second hexadecimal number once
convert_decimal:
	PUSH R1
	PUSH R2
	PUSH R3
	PUSH R4
	MOV  R2, R9               ; R2 <- result value
	MOV  R1, R9               ; R1 <- iterator
	MOV  R3, 10               ; 0x000A (10)
	MOV  R4, 6                ; 0x0006 (6)
	SUB  R1, R3               ; R1 < 10, no action, else, convert  
	JN   convert_decimal_out
first_hex_digit:
	ADD  R2, R4               ; add 6 every iteration 
	SUB  R1, R3               ; next iteration, remove 10
	JNN  first_hex_digit
	SHL  R3, 4                ; 0x00A0 (160,but acts like 100 in decimal would) 
	SHL  R4, 4                ; auxiliary
	CMP  R3, R2               ; if R2 (current decimal representation) < 100, no action, else, convert 
	JP   convert_decimal_out
	ADD  R2, R4               ; add 60 if if R2 >= 100
convert_decimal_out:
	MOV  [P_OUT1], R2         ; write to displays
end_convert:
	CMP  R9, 0                ; if R9 -> 0, ran out of energy, end game
	JNZ  valid
	MOV  R1, 1
	MOV [IS_PAUSED], R1       ; set IS_PAUSED flag to 1 -> stop processes
	MOV [END], R1             ; set END flag to 1 -> signals end of game
	MOV R1, END_IMAGE         ; write end image to display
	MOV [FRONT_MEDIA], R1
valid:
	POP  R4
	POP  R3
	POP  R2
	POP  R1
	RET

;*****************************************************************************************;
;
;                                  OBJECTS PROCESS
;
;*****************************************************************************************;
PROCESS SP_objects_update
objects_update:
	MOV  R1, [INT_0]          ; LOCK process if paused
	MOV  R1, [IS_PAUSED]      ; check global status variable
	CMP  R1, 1                ; if 1, game is paused
	JZ   objects_update

; check for previous explosion to clear
	MOV  R2, EXPLOSION_COORDS
	MOV  R0, [R2]
	CMP  R0, 0               ; if explosion coordinates are not null, explosion happened 
	JZ   no_explosion
	MOV  R0, SCREEN_1        ; media center screen argument
	MOV  R1, EXPLOSION       ; explosion object descriptor argument
	CALL erase_object        ; erase explosion
; determine new object form
no_explosion:
	MOV  R0, SCREEN_1        ; mediacenter screen argument
	MOV  R2, OBJECTS_COORDS  ; enemy coordinates argument
	CALL determine_form      ; determine form of object
	CALL erase_object        ; erase current object
	MOV  R3, [R2 + 2]        ; R3 <- current Y object coordinate
	INC  R3                  ; increment object Y coordinate
	MOV  [R2 + 2], R3        ; update new Y coordinate

; check for collisions
	MOV  R0, MAX_LINE        ; max number of lines (max Y coordinates)
	CMP  R3, R0              ; if disappearing through the screen -> new object 
	JLT  check_missile_meteor
	CALL new_object          ; randomize new object and reset Y coordinate
	JMP  end_meteor
check_missile_meteor:
	MOV  R4, RANGE         ; shooting range
	MOV  R5, MAX_LINE
	SUB  R5, R4            ; Y where object is in shooting range
	MOV  R6, [R1]          ; object height
	SUB  R5, R6
	SUB  R5, 4
	CMP  R3, R5                       ; if object is in shooting range -> detect collision
	JLT  check_spacerover_collision
	CALL missile_collision            ; check for collision, returns value on R8
	CMP  R8, 1                        ; if object collided with missile -> handle it, else check spacerover collision
	JNZ  check_spacerover_collision
	CALL handle_missile_collision     ; handle collision
	CALL new_object                   ; calculate new object
	JMP end_meteor
check_spacerover_collision:
	MOV  R4, SPACEROVER         ; spacerover descriptor address
	MOV  R5, MAX_LINE
	MOV  R6, [R4]               ; spacerover height
	SUB  R5, R6                 ; R5 <- Y of rover's top pixels
	MOV  R2, OBJECTS_COORDS     ; object coordinates
	MOV  R3, [R2 + 2]           ; R3 <- object Y coordinate
	ADD  R3, 4                  ; add height
	CMP  R3, R5                 ; if object Y is in collision w spacerover range
	JLT  draw
	CALL spacerover_collision   ; check spaceship collision, returns result in R7
	CMP  R7, 0                  ; if there is a collision, dont draw object
	JNZ  end_meteor
draw:
	CALL draw_object
end_meteor:
	JMP	 objects_update

;********************************************************************************
; Determines the form of an object (5x5, 4x4, etc) depending on its coordinates and type
; Arguments:
; R2 - Object coordinates
; Returns:
; R1 - Object descriptor
;********************************************************************************
determine_form:
	PUSH R0
	PUSH R2
	PUSH R3
	PUSH R4
	MOV  R3, ENEMY_ROVER_FORMS
	MOV  R0, [R2 + 4]          ; check if it's a meteor or an enemy airrover
	CMP  R0, 0                 ; if 0 it's a meteor, else it's an airrover
	JNZ  determine_size
	MOV  R3, METEOR_FORMS
determine_size:
	MOV  R0, [R2 + 2]          ; current line of object
	MOV  R4, 3                 ; divisor ( object changes form every 3 lines  )
	DIV  R0, R4                
	CMP  R0, 5                 ; if line // 3 = 0 -> first form, if 1 -> second form and so on untill 5th form
	JLT  normal
	MOV  R0, 4                 ; if line // 3 > 5, it's already in its final form
normal:
	SHL  R0, 1                 ; multiply form value by 2, because each descriptor address occupies 2 bytes
	MOV  R1, [R3 + R0]         ; R1 <- address of current object's form descriptor
	POP  R4
	POP  R3
	POP  R2
	POP  R0
	RET
	
;********************************************************************************
; Randomizes a new object and resets its coordinates 
; Arguments:
; R2 - Object coordinates
;********************************************************************************
new_object:
	PUSH R0
	PUSH R1
	;calculate new coordinates
	MOV  R0, PIN
	MOVB R1, [R0]              ; random 4-7 bits
	SHR  R1, 5                 ; shift 5-7 bits to 0-2 bits position (random digit from 0 to 7 is stored in R1)
	SHL  R1, 3                 ; multiply by 8 (randomize one out of 8 possible columns)
	ADD  R1, 2                 ; add 2 to avoid objects hugging the broder
	MOV  [R2], R1              ; update object X coordinate
	MOV  R1, 0                 ; update object Y coordinate (reset)
	MOV  [R2 + 2], R1
	; calculate new form
	MOVB R1, [R0]              ; random 4-7 bits
	SHR  R1, 6                 ; shift 6-7 bits to 0-1 bits position (random digit from 0 is 4 is stored in R1)
	CMP  R1, 0                 ; if 0 then new object is a meteor, else it is an enemy spacerover ( 25 % chance of meteor)
	JZ   meteor
	ADD  R1, 1                 ; if it is an enemy spacerover then form = 1
meteor:
	MOV  [R2 + 4], R1          ; update new object form
	POP  R1
	POP  R0
	RET

;********************************************************************************
; Checks if there is a collision with the missile and the given object 
; Arguments:
; R1 - Object descripor
; R2 - Object coordinates
; Returns:
; R8 - 1 if there is a collision, else 0
;********************************************************************************
missile_collision:
	PUSH R0
	PUSH R1
	PUSH R2
	PUSH R4
	PUSH R5
	PUSH R6
	; check missile collision
	MOV  R8, 0               ; default return value
	MOV  R0, [FIRE]          ; FIRE flag
	CMP  R0, 0               ; if no missile is fired don't check for missile collision
	JZ   end_collision
	MOV  R0, MISSILE_COORDS  ; missile coordinates address
	MOV  R3, [R2]            ; R3 <- left side pixel X of object
	MOV  R6, [R1 + 2]        ; object width
	MOV  R4, R3
	ADD  R4, R6              ; left side pixel + width 
	DEC  R4                  ; R4 <- right side pixel X of object 
	MOV  R5, [R0]            ; R5 <- bullet X coordinate
	CMP  R5, R3              ; if bullet X coordinate < left side pixel X -> no collision
	JLT  end_collision
	CMP  R5, R4              ; if bullet X coordinate > right side pixel X -> no collision 
	JGT  end_collision
	MOV  R4, [R2 + 2]        ; object Y coordinate
	MOV  R5, [R1]            ; object height
	ADD  R4, R5              ; add height to top corner pixel
	DEC  R4                  ; R4 <- Y coordinate of object bottom pixels
	MOV  R3, [R0 + 2]        ; R3 <- bullet Y coordinate
	DEC  R3
	CMP  R3, R4              ; if missile Y coordinate >= object Y bottom pixels coord -> collision
	JGT  end_collision
	MOV  R8, 1
end_collision:
	POP R6
	POP R5
	POP R4
	POP R2
	POP R1
	POP R0
	RET
	
;********************************************************************************
; Checks if there is a collision with the spacerover and the given object 
; Arguments:
; R1 - Object descripor
; R2 - Object coordinates
; Returns:
; R7 - 1 if there is a collision, else 0
;********************************************************************************
spacerover_collision:
	PUSH R1
	PUSH R2
	PUSH R3
	PUSH R4
	PUSH R5
	PUSH R6
	MOV  R8, R1
	MOV  R9, R2
	MOV  R7, 0
	; check missile collision
	MOV  R3, [R2]            ; object X
	MOV  R4, [R2 + 2]        ; object Y
	MOV  R5, [R1]            ; object height
	MOV  R6, [R1 + 2]        ; object width
	MOV  R1, R3              ; R1 <- left side object pixel X              
	MOV  R2, R1              ; R2 <- right side object pixel X
	ADD  R2, R6              ; add width to pixel 
	DEC  R2                  ; make R2 be the X coordinate of the right side pixel
	MOV  R5, SPACEROVER_COORDS
	MOV  R3, [R5]            ; R3 <- left side rover pixel X
	MOV  R5, SPACEROVER 
	MOV  R6, [R5 + 2]        ; object width
	MOV  R4, R3              ; R4 <- right side rover pixel X
	ADD  R4, R6              ; add width to left side pixel X
	DEC  R4                  ; make R4 be the X coordinate of the right side pixel of rover
	CMP  R4, R1              ; if right side of rover X < left side of object X -> no collision
	JLT  end_collision_2 
    CMP	 R3, R2              ; if left side of rover X > right side of object X -> no collision
	JGT  end_collision_2
	MOV R1, R8
	MOV R2, R9
	CALL handle_spacerover_collision  ; handle collision 
	MOV  R7, 1
end_collision_2:
	POP R6
	POP R5
	POP R4
	POP R3
	POP R2
	POP R1
	RET

;********************************************************************************
; Handles collision of a missile with an object 
; Arguments:
; R1 - Object descripor
; R2 - Object coordinates
;********************************************************************************
handle_missile_collision:
	PUSH R0
	PUSH R1
	PUSH R2
	PUSH R8
	PUSH R9
	; determine if enemy rover exploded
	MOV  R0, [R2 + 4]          ; object type, if 1 -> enemy rover, else meteor
	CMP  R0, 0                 ; if enemy rover was hit update energy
	JZ   draw_explosion
	MOV  R9, HIT_ROVER_VALUE   ; display increment argument
	MOV  R8, ADD_OP            ; operation identifier argument
	CALL update_display        ; update displays and energy counter
draw_explosion:
	MOV  R0, EXPLOSION_COORDS      ; update explosion coordinates
	MOV  R1, [R2]                  ; object X coordinate
	MOV  [R0], R1                  ; update explosion explosion X
	MOV  R1, [R2 + 2]              ; 0bject Y
	MOV  [R0 + 2], R1              ; update explosion explosion Y
	MOV  R0, 0
	MOV  [FIRE], R0                ; signal end of fire event
	MOV  R0, SCREEN_1              ; mediacenter screen argument
	MOV  R1, MISSILE               ; missile object descriptor argument
	MOV  R2, MISSILE_COORDS        ; missile coordinates argument
	CALL erase_object              ; erase bullet
	MOV  R0, SCREEN_1              ; mediacenter screen argument
	MOV  R1, EXPLOSION             ; explosion object descriptor argument
	MOV  R2, EXPLOSION_COORDS      ; explosion coordinates argument
	CALL draw_object               ; draw explosion
	MOV  R0, EXPLOSION_SOUND
	MOV  [PLAY_SOUND], R0          ; play explosion sound

	POP  R9
	POP  R8
	POP  R2
	POP  R1
	POP  R0
	RET

;********************************************************************************
; Handles collision of an object with the spacerover 
; Arguments:
; R1 - Object descripor
; R2 - Object coordinates
;********************************************************************************
handle_spacerover_collision:
	PUSH R3
	MOV  R3, [R2 + 4]    ; object type ( 1 - spacerover, 0 meteor)
	CMP  R3, 0           ; if 0, increment energy, else end game
	JNZ  end 
	MOV  R9, GOOD_METEOR_VALUE  ; energy increment value for meteor collision argument
	MOV  R8, ADD_OP             ; operation argument for update_display
	CALL update_display
	MOV  R0, SCREEN_1           ; media center screen argument
	CALL erase_object           ; erase object
	CALL new_object             ; recalculate object in new coordinates
	MOV R3, GOOD_METEOR_SOUND
	MOV [PLAY_SOUND], R3        ; play good meteor hit sound
	JMP  leave_col
end:
	MOV R3, 1
	MOV [IS_PAUSED], R3           ; stop all processes
	MOV [END], R3                 ; set END flag to 1
	MOV R3, ROVER_EXPLODE_IMAGE
	MOV [FRONT_MEDIA], R3         ; explosion image
	MOV R3, EXPLOSION_SOUND
	MOV [PLAY_SOUND], R3          ; explosion sound
leave_col:
	POP R3
	RET

;******************************************************************************
;*
;*                           KEYBOARD PROCESS 
;*
;******************************************************************************
; This process writes to KB_KEY and CONT_KB_KEY a number where the bits 0-3 store the information of the line
; and 4-7 the informtation of the column of the key
; If the key is being held, the keyboard starts only writting to CONT_KB_KEY untill the key is released
PROCESS SP_keyboard
keyboard:
	MOV  R1, 1         ; first line of keyboard
loop:
	
	YIELD

	MOV  R3, P_OUT2    ; keyboard lines address
	MOVB [R3], R1      ; feed current line of keyboard 
	MOV  R3, PIN       ; keyboard columns memory address
	MOVB R2, [R3]      ; get column output result 
	MOV  R4, MASK      ; 0x000F
	AND  R2, R4        ; only account 4 least significant bits
	JNZ  handle        ; handle click
	SHL  R1, 1         ; check next line
	AND  R1, R4        ; line number AND 0x000F = 0 -> last line checked already, else, check next line
	JNZ  loop
	JMP  keyboard

handle:
	MOV  R4, 1010b
	AND  R4, R1        ; line nr AND 0b1010 = 0 -> 1st or 3rd line clicked, unused lines
	JZ   keyboard
	MOV  R4, 0111b
	AND  R4, R2        ; column nr AND 0b0111 = 0 -> 4th column clicked, unused column
	JZ   keyboard
	MOV  R0, R2
	SHL  R0, 4         ; 4-7 bits store column information
	ADD  R0, R1        ; 0-3 bits store line information
	MOV  [KB_KEY], R0  ; unlocks processes that read LOCK 


	MOV  R1, MASK   ; 0x000F
	MOV  R2, PIN    ; keyboard columns address
hold_key:

	YIELD

	MOV  [CONT_KB_KEY], R0  ; while there is a key being pressed
	MOVB R3, [R2]           ; R2 <- keyboard columns return value
	AND  R3, R1             ; keep reading from columns until they are zero (key is released)
	JNZ  hold_key

	JMP  keyboard 
	
;******************************************************************************
;*
;*                           START PROCESS 
;*
;******************************************************************************
PROCESS SP_start
start:	
	MOV   R0, [KB_KEY]       ; LOCK processes untill keyboard key is clicked
	MOV   R1, 1000b          ; 1000b (8)
	AND   R1, R0             ; checks if the key is on the 4th line	      
	JZ    start
	SHR   R0, 4               ; ignore line info, get 0-3 bits with column info 
	MOV   R1, 0001b           ; 0001b (1)
	AND   R1, R0	          ; checks if the key is the C key
	JZ    start

	MOV   R9, MAX_ENERGY        ; argument for convert_decimal rotine
	CALL  convert_decimal       ; changes the display to MAX_ENERGY
	MOV   [ENERGY_COUNTER], R9  ; reset energy counter

	MOV   R2, OBJECTS_COORDS    ; objects coordinates argument
	CALL  determine_form        ; determine form of objects to clean from screen
	CALL  erase_object          ; clean objects from screen
	MOV   R1, MAX_LINE 
	MOV   [R2 + 2], R1          ; put object Y cooridnate at 32 (objects process will randomize them in the next run)

	MOV   R0, SCREEN_0           ; media center screen argument
	MOV   R1, SPACEROVER         ; spacerover object descriptor
	MOV   R2, SPACEROVER_COORDS  ; spacehrover coordinates
	CALL  erase_object           ; erase rover from screen

	MOV   R1, ROVER_INIT_X       ; 32
	MOV   [R2],R1                ; reset spacerover X coordinate
	MOV   R1, SPACEROVER
	CALL  draw_object            ; redraw the spacerover in the middle

	MOV   R1, 0
	MOV   [END], R1              ; reset END flag to 0
	MOV   [IS_PAUSED], R1        ; reset PAUSE flag to 0
	MOV   [FIRE], R1             ; reset FIRE flag to 0
	MOV   [DEL_FRONT_MEDIA], R2  ; delete pause message

	MOV   R0, SCREEN_1           ; mediascreen screen argument
	MOV   R1, MISSILE            ; missile object descriptor argument
	MOV   R2, MISSILE_COORDS     ; missile coordinates argument
	CALL  erase_object           ; delete a missile that is midair
	JMP   start


;********************************************************************************
;*                        GENERAL PURPOSE ROUTINES
;********************************************************************************
;********************************************************************************
;* Draws given object descriptor in MediaCenter's screen passed as argument
;* Arguments: 
;* R0: MediaCenter screen number
;* R1: Adress of object descriptor
;* R2: Adress of object coordinates
;********************************************************************************
draw_object:
	PUSH R3
	PUSH R4
	PUSH R5
	PUSH R6
	PUSH R7
	PUSH R8
	MOV  [SEL_SCREEN], R0  ; select screen
	MOV  R8, MAX_LINE      ; used to detect disappearing through the bottom of the screen 
	MOV  R4, [R1]          ; R4 <- object height
	MOV  R5, [R2 + 2]      ; R5 <- current pixel Y coordinate
	MOV  R0, R1            ; R0 <- copy of object descriptor address used to iterate
	ADD  R0, 4             ; skip height and width addresse
draw_loop1:
	MOV  R6, [R2]          ; R6 <- current pixel X coordinate
	MOV  R3, [R1 + 2]      ; R3 <- object width
	CMP  R5, R8            ; current pixel Y coordinate >= MAX_LINE -> object disappearing
	JGE  draw_ret
	MOV  [LINE], R5        ; select line 
draw_loop2:
	MOV  [COLUMN], R6      ; select column
	MOV  R7, [R0]          ; current pixel value
	MOV  [PIXEL], R7       ; draw pixel
	ADD  R0, 2             ; next pixel descriptor
	INC  R6                ; next column
	DEC  R3                ; decrement column iterator
	JNZ  draw_loop2
	INC  R5                ; next line
	DEC  R4                ; decremnt line iterator
	JNZ  draw_loop1
draw_ret:
	POP  R8
	POP  R7
	POP  R6
	POP  R5
	POP  R4
	POP  R3
	RET

;********************************************************************************
;* Erases given object descriptor in MediaCenter's screen passed as argument
;* Arguments: 
;* R0: MediaCenter screen number
;* R1: Adress of object descriptor
;* R2: Adress of object coordinates
;********************************************************************************
erase_object:
	PUSH R3
	PUSH R4
	PUSH R5
	PUSH R6
	PUSH R7
	PUSH R8
	MOV  [SEL_SCREEN], R0  ; select screen
	MOV  R8, MAX_LINE      ; detect disappearing on the bottom
	MOV  R7, BLANK         ; 0x0000, trasparent color (pratically deleting pixel)
	MOV  R4, [R1]          ; R4 <- object height
	MOV  R5, [R2 + 2]      ; R5 <- current pixel Y coordinate
erase_loop1:
	MOV  R3, [R1 + 2]      ; R3 <- object width
	MOV  R6, [R2]          ; R6 <- current pixel X coordinate
	CMP  R5, R8            ; current pixel Y coordinate >= MAX_LINE -> object disappearing
	JGE  erase_ret
	MOV  [LINE], R5        ; select line 
erase_loop2:
	MOV  [COLUMN], R6      ; select column
	MOV  [PIXEL], R7       ; draw pixel
	INC  R6                ; next column
	DEC  R3                ; decrement column iterator
	JNZ  erase_loop2
	INC  R5                ; next line
	DEC  R4                ; decrement line iterator
	JNZ  erase_loop1
erase_ret:
	POP  R8
	POP  R7
	POP  R6
	POP  R5
	POP  R4
	POP  R3
	RET

;********************************************************************************
;* Initialize media center
;********************************************************************************
setup:
	; table to store forms of objects
	MOV  R2, METEOR_FORMS               ; table with address of all meteor forms descriptors
	MOV  R3, ENEMY_ROVER_FORMS          ; table with address of all enemy rover form descriptors

	; fill tables at setup
	MOV  R1, OBJECT_1
	MOV  [R2], R1
	MOV  [R3], R1
	MOV  R1, OBJECT_2
	MOV  [R2 + 2], R1
	MOV  [R3 + 2], R1
	; meteors
	MOV  R1, METEOR_3
	MOV  [R2 + 4], R1
	MOV  R1, METEOR_4
	MOV  [R2 + 6], R1
	MOV  R1, METEOR_5
	MOV  [R2 + 8], R1
	; enemy rovers
	MOV  R1, ENEMY_ROVER_3
	MOV  [R3 + 4], R1
	MOV  R1, ENEMY_ROVER_4
	MOV  [R3 + 6], R1
	MOV  R1, ENEMY_ROVER_5
	MOV  [R3 + 8], R1

	; initialize media center
	MOV  [REMOVE_WARNING], R0
	MOV  [SEL_BACKGROUND], R0

	; draw spacerover
	MOV  R0, SCREEN_0              ; media screen argument
	MOV  R1, SPACEROVER            ; spacerover descriptor argument
	MOV  R2, SPACEROVER_COORDS     ; rover coordinates argument
	CALL draw_object

	; initial energy
	MOV  R9, MAX_ENERGY
	MOV  [ENERGY_COUNTER], R9
	CALL convert_decimal

	; display start image
	MOV R1, START_IMAGE
	MOV [FRONT_MEDIA], R1 
	RET


;********************************************************************************
;*                        INTERRUPTIONS
;********************************************************************************
int0:
	PUSH R0
	MOV  R0, 1
	MOV  [INT_0], R0       ; unlock objects process
	POP  R0
	RFE

int1:
	PUSH R0
	MOV  R0, 1
	MOV  [INT_1], R0      ; unlock fire process
	POP  R0
	RFE

int2:
	PUSH R0
	MOV  R0, 1
	MOV  [INT_2], R0      ; unlock energy process
	POP  R0
	RFE
	
