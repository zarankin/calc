
section .bss
    string: resb 82
    linkkk: resb 4
    link2: resb 4
    link3: resb 4
    link4: resb 4
    lastLink:resb 4
    firstLink:resb 4
section .data
    myTable: db 0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15
    mulStack: times 32 db 0
    myStack: times 40 db 0
	format_int: db "%X", 10, 0	; format int
   	format_string: db "%s", 10, 0	; format string
    format_string2: db "%s", 0	; format string
    error1: db "Error: Insufficient Number of Arguments on Stack",0
	error2: db "Error: Operand Stack Overflow",0
	error3: db "wrong Y value",0
    PRcalac: db "calc: ",0
	rowDown: db 0x0a
	tmp: dd 0
	first: db 1
	char: db 0
	carryToAdd:db 0
	forClean: dd 0
	tmpCounter:dd 0
    numOflinks: dd 0
    capacity: dd 0
	operetions: dd 0
    debug: db 0
    rootCount:dd 0


    section .text
	align 16
	global main 
	extern printf 
	extern fflush
	extern malloc 
	extern calloc 
	extern free 
	extern gets 
main: 
push ebp
mov ebp,esp
sub esp,8
mov edi, dword[ebp+8]
cmp edi,2
jz dM

mov byte[debug],0
jmp myCalc

dM:
mov esi,dword[ebp+12]
mov ebx,dword[esi+4]
mov eax,dword[ebx]
cmp ax,"-d"
jnz myCalc
mov byte[debug],1
jmp myCalc


myCalc:

input:
    pushad
    push PRcalac
    push format_string2
    call printf
    push dword 0
    call fflush
    add esp,12

    pushad
	push dword string
	call gets
	add esp, 4
	popad


clean:
    mov byte[carryToAdd],0
    mov dword[tmpCounter],0
    mov byte[first],1
    mov dword[forClean],0
	xor ecx,ecx
	xor eax,eax
	xor ebx, ebx
	xor edx,edx

checking:
	mov edx,string			          ;ebx hold pointer to input string
	cmp byte[edx], 0x71		          ;compare with q
	jz exiting

	cmp byte[edx], 0x2B		          ;compare with + 
	jz plus

	cmp byte[edx], 0x70		          ;compare with p
	jz popNprint

	cmp byte[edx], 0x64		          ;compare with d
	jz duplicate

	cmp byte[edx], 0x5E		          ;compare with ^
	jz power

	cmp byte[edx], 0x76		          ;compare with v
	jz powerNeg

	cmp byte[edx], 0x6E		          ;compare with n
	jz countOne

	cmp byte[edx], 0x73		          ;compare with s
	jz squareRoot

	jmp pushNum


alocation:
        pushad
        push dword 9                    ;we want the every link will be 9 bytes
        call malloc
        mov dword[tmp],eax      
        add esp,4                       ;clean what we pushed
        popad     
        ret                      


freeing:
        pushad
        push ecx
        call free
        add esp,4
        popad
        ret

moveCapacity:
        mov ebx, dword[capacity]               ;save the capacity in ebx
        dec ebx                                ;we need the cell in place capacity-1
        imul ebx,8                             ;we need 12 junmps because every cell is 12 bytes 
        ret

lineFree:
        mov eax,dword[ecx+1]                   ;saving in eax the adress of the next link
        call freeing
        mov ecx,eax
        cmp ecx,0
        jnz lineFree
        ret

debugMode:
    call moveCapacity
    mov ecx,dword[myStack+ebx]              ;save the address of the first link in ecx

    
.loop:
    mov al,byte[ecx]
    inc ecx                                 ;at place link+1 we have the adress of next link
    mov edx,dword[ecx]                      ;saving in edx the adress of the next link

    dec ecx 

    cmp al,10                           ;checking if we need to add 55 or 48
    jnl .upTOten
    jmp .lessThenTen
    
    .upTOten:
        add al, 55
        jmp .pr
        
    .lessThenTen:
        add al, 48
        jmp .pr
        
.pr:
    mov byte[char],al                   ;saving the current char for printing

    pushad                              
    mov eax, 4
    mov ebx,1
    mov ecx,char
    mov edx,1
    int 0x80
    popad

    mov ecx,edx
    cmp ecx,0
    jz .return

    jmp .loop
    
.return:
    pushad
    mov ecx, rowDown                    ;printing /n
    mov eax,4
    mov ebx,1
    mov edx,1
    int 0x80
    popad
    jmp input


exiting:
	push dword[operetions]
	push format_int
	call printf
	push dword 0
	call fflush
	add esp, 12			                ;cleaned last 2 push

    .run:
    cmp dword[capacity],0               ;checking if there is more cells
    jz .end
    call moveCapacity                   ;ebx=capacity-1

    mov ecx,dword[myStack+ebx]          ;ecx=first link
    pushad
    call lineFree
    popad
    dec dword[capacity] 
    jmp .run

.end:
	mov eax, 1
	mov ebx,0
	int 0x80

popNprint:
    cmp byte[capacity],1
    jl errorPrint1
    inc dword[operetions]
    
    call moveCapacity
    mov ecx,dword[myStack+ebx]          ;save the address of the first link in ecx
    
    mov eax,dword[ecx+1]                ;saving in eax the adress of the next link
    mov dword[linkkk],eax               ;save in linkkk the next link
    jmp .poping
    
    
.poping:
    
    xor eax,eax
    mov al,byte[ecx]                    ;saving the data in al
    
    cmp al,10                           ;checking if we need to add 55 or 48
    jnl .upTOten
    jmp .lessThenTen
    
    .upTOten:
        add al, 55
        jmp .pr
        
    .lessThenTen:
        add al, 48
        jmp .pr
        
.pr:
    mov byte[char],al                   ;saving the current char for printing

    pushad                        
    call freeing                           ;free alocated space
    mov eax, 4
    mov ebx,1
    mov ecx,char
    mov edx,1
    int 0x80
    
    popad
    mov ecx,dword[linkkk]               ;save at ecx the next link
    cmp ecx,0                           ;checking if we finished the linked list
    jz .retToInput
    mov eax,dword[ecx+1]                 ;saving the address at eax
    mov dword[linkkk],eax               ;and now in linkkk
    jmp .poping
    
.retToInput:
    dec dword[capacity]
    pop eax
    pushad
    mov ecx, rowDown                    ;printing /n
    mov eax,4
    mov ebx,1
    mov edx,1
    int 0x80
    popad
    jmp input


duplicate:
    cmp byte[capacity],1                ;jump if less
    jl errorPrint1
    
    cmp byte[capacity],5                ;jump if 0
    jnz .con
    
    inc dword[operetions]
    jmp errorPrint2

    .con:
    inc dword[operetions]
    call moveCapacity
    mov ecx,dword[myStack+ebx]          ;save the address of the first link in ecx
    inc dword[capacity]
    
    mov eax,dword[ecx+1]                  ;saving in eax the adress of the next link
    mov dword[linkkk],eax               ;save in linkkk the next link
    
    call moveCapacity
    .copying:
    
        call alocation
        mov edx, dword[tmp]             ;edx now a pointer to alocated memory
        xor eax,eax
        mov al,byte[ecx]                ;saving the data in al
        mov byte[edx], al               ;copy the data to edx[0]
        mov dword[edx+1],0                ;in cell 1-4 we save the next link
        
        cmp byte[first],1               ;checking the flag first
        jz .firstPush
        
        push ebx
        mov ebx,dword[link2]            ;saving in ebx the adress of prev link
        mov dword[ebx+1],edx            ;saving the adress of curr as next in the prev link
        mov dword[edx+5],ebx            ;in cell 5-8 we save the prev link
        pop ebx                         ;return ebx to capacity value

        mov dword[link2],edx            ;saving the link
        
        mov ecx,dword[linkkk]           ;save at ecx the next link
        cmp ecx,0                       ;checking if we finished the linked list
        jz .retToInput
        mov eax,dword[ecx+1]            ;saving the address at eax
        mov dword[linkkk],eax           ;and now in linkkk
        jmp .copying
        
    .firstPush:

        mov dword[myStack+ebx],edx      ;save the first link at stack+capacity
       
        pushad
        mov byte[first],0               ;changing the flag
        mov eax,myStack
        add eax, ebx
        mov dword[edx+5],eax            ;saving the prev in the link(the prev is the stack)
        mov dword[link2],edx            ;saving the link
        popad
        
        mov ecx,dword[linkkk]           ;save at ecx the next link
        cmp ecx,0                       ;checking if we finished the linked list
        jz .retToInput
        mov eax,dword[ecx+1]              ;saving the address at eax
        mov dword[linkkk],eax           ;and now in linkkk
        jmp .copying
    
    .retToInput:
        xor edx,edx
        mov edx,dword[link2]
        add ebx,4
        mov dword[myStack+ebx],edx
        cmp byte[debug],1
        jz debugMode
        jmp input

countOne:
    cmp byte[capacity],1
    jl errorPrint1

    inc dword[operetions]
    
    call moveCapacity
    mov ecx,dword[myStack+ebx]          ;save the address of the first link in ecx
    mov dword[linkkk],ecx               ;saving to delete in the end
    
    jmp .counting
    
    
.counting:
    
    cmp ecx,0                           ;checking if the end of the links
    jz .retToInput
    xor eax,eax
    xor edx,edx
    mov al,byte[ecx]                    ;saving the data in al
    
    mov dl,2
    call .pcnt
    
    xor dl,dl

    cmp byte[first],1
    jnz .con
    call .firstPush
    .con:
    add byte[ebx],dh                   ;adding the number to the counter
        
    cmp byte[ebx],16                    ;checkig if need new link
    jnl .adding
    
    mov ecx,dword[ecx+1]                ;save at ecx the next link
    cmp ecx,0                           ;checking if we finished the linked list
    jz .retToInput
    jmp .counting
    

.firstPush:
    call alocation
    mov eax, dword[tmp]

    mov byte[first],0
    mov byte[eax],0
    mov dword[eax+1],0                  ;next
    mov dword[eax+5],0                  ;prev
    mov ebx,eax
    mov dword[link2],eax
    ret

.pcnt:
div dl
add dh,ah
xor ah,ah

div dl
add dh,ah
xor ah,ah

div dl
add dh,ah
xor ah,ah

div dl
add dh,ah
xor ah,ah

ret

.adding:
    pushad
    .myloop:
    cmp dword[ebx+5],0                      ;checking the prev
    jnz .con2 
    call .create
.con2:
    sub byte[ebx],16
    mov ebx,dword[ebx+5]
    add byte[ebx],1
    cmp byte[ebx],16
    jz .myloop
    popad
    mov ecx,dword[ecx+1]
    jmp .counting

.create:
    call alocation
    mov eax,dword[tmp]

    mov byte[eax],0
    mov dword[eax+1],ebx
    mov dword[eax+5],0

    mov dword[ebx+5],eax
    mov dword[link2],eax                ;saving the first link
    ret


.retToInput:
    mov eax,dword[link2]                ;first=the last one that created
    mov edx,ebx
    call moveCapacity
    mov dword[myStack+ebx],eax          ;first
    mov ecx,myStack
    add ecx,ebx
    mov dword[eax+5],ecx
    add ebx,4
    mov[myStack+ebx],edx                ;last link

    mov ecx,dword[linkkk]
    call lineFree

    cmp byte[debug],1
    jz debugMode
    jmp input


plus:
	cmp byte[capacity],2
	jl errorPrint1
	
	mov byte[carryToAdd],0
	inc dword[operetions]
	

    call moveCapacity
    add ebx,4
    mov edx,[myStack+ebx]               ;last link of x
    
    dec dword[capacity]
    call moveCapacity
    add ebx,4
    mov ecx, dword[myStack+ebx]         ;last link of y
    
    .adding:
        xor ebx,ebx
        xor eax,eax
        
        mov al, byte[ecx]               ;saving the number that we need to add in al from ecx
        shl al,4                        ;because hex value is only 4 bits, we need in to fill 8 bits for the carry flag
        mov bl, byte[edx]
        shl bl,4
        add al,bl                       ;adding and checking the carry flag
        jc .haveC
        jmp .noC
        
     .haveC:
        add al,byte[carryToAdd]         ;adding to al the prev carry
        mov byte[carryToAdd],16         ;saving in carryToAdd the new carry
        shr al,4                        ;return to right num
        jmp .con 
    
        .moveCarry:
        mov byte[carryToAdd],16
        shr al,4                        ;return to right num
        jmp .con

    
    .noC:
        add al, byte[carryToAdd]        ;adding the prev carry
        jc .moveCarry                   ;chacking if now we have carry
        shr al,4                        ;return to right num
        mov byte[carryToAdd],0          ;saving the new carry
        jmp .con
        
        
    .con:
        mov byte[ecx],al
        call moveCapacity
        mov eax,myStack
        add eax,ebx
        cmp eax, dword[ecx+5]
        jz .endY

        inc dword[capacity]
        call moveCapacity
        mov eax, myStack
        add eax,ebx
        cmp eax,dword[edx+5]
        jz .endX
        
        dec dword[capacity]
        mov ecx,dword[ecx+5]
        mov edx,dword[edx+5]
        
        jmp .adding
        
    .endY:
        mov byte[first],0
        inc dword[capacity]
        call moveCapacity
        mov eax, myStack
        add eax,ebx
        cmp eax,dword[edx+5]
        jz .endX

        pushad
        dec dword[capacity]
        call alocation
        mov eax,dword[tmp]
        mov dword[ecx+5],eax
        mov byte[eax],0
        mov dword[eax+1],ecx
        popad

        mov ecx,dword[ecx+5]
        call moveCapacity
        mov eax, myStack
        add eax,ebx
        mov dword[ecx+5],eax
        mov dword[myStack+ebx],ecx
        mov edx,dword[edx+5]
        jmp .adding
        
    .endX:
        dec dword[capacity]
        cmp byte[first],0
        jz .finishedHere



        .scon:
        mov byte[edx],0
        mov ecx,dword[ecx+5]
        jmp .adding 
        
    .finishedHere:
        xor eax,eax
        
        mov al,byte[carryToAdd]         ;saving the carry in al to check if we need another link
        cmp al,0
        jz .conn
        
        call alocation
        mov eax,dword[tmp]
        mov dword[ecx+5],eax
        mov byte[eax],1
        mov dword[eax+1],ecx

        mov ecx,dword[ecx+5]
        call moveCapacity
        mov eax, myStack
        add eax,ebx
        mov dword[ecx+5],eax
        mov dword[myStack+ebx],ecx
    .conn:
        mov ecx,edx
        call lineFree
        
        cmp byte[debug],1
        jz debugMode
        jmp input
        

power:
     cmp byte[capacity],2                ;checking if there is less the 2 numbersm in the stack
    jl errorPrint1
    
    mov ebx,dword[capacity]             ;capacity at least 2
    sub ebx,2                           ;need the prev cell in the stack
    imul ebx,8
    
    mov ecx,dword[myStack+ebx]          ;ecx=y
    mov al,byte[ecx]
    mov ecx,dword[ecx+1]

    cmp ecx,0
    jz .con
    shl al,4

    xor edx,edx

    add al,byte[ecx]
    mov dl,200
    cmp ax,dx                           ;checking if y is more then 200
    jg errorPrint3

    mov ecx,dword[ecx+1]                ;if there is more then 2 links its more then 200
    cmp ecx,0
    jnz errorPrint3

    .con:
    inc dword[operetions]
    xor edx,edx
    mov dl,4
    div dl

    call moveCapacity
    add ebx,4
    mov ecx,dword[myStack+ebx]          ;last link of x
    mov dword[link2],ecx

    .myLoop:
    cmp al,0
    jz .shifting

    call alocation
    mov edx,dword[tmp]
    mov byte[edx],0
    mov dword[edx+1],0
    mov dword[edx+5],ecx

    mov dword[ecx+1],edx
    mov ecx,edx
    dec al
    jmp .myLoop


    .shifting:
    call moveCapacity
    xor edx,edx
    add ebx,4
    mov dword[myStack+ebx],ecx
    sub ebx,4
    add ebx,myStack

    cmp ah,0
    jz .finished
    mov ecx,dword[link2]
    push ecx
    xor ecx,ecx
    mov cl,ah
    mov dl,2
    mov al,1

    .smallLoop:
    mul dl
    loop .smallLoop,ecx

    pop ecx
    mov dl,al

    .loopy:
    mov al, byte[ecx]                   ;saving in eax the data that saved in the link
    shl al,4                            ;doing shift to catch the carry
    mul dl                              ;mul the data in 2^edx=>the same as doing shift but saving the bites that could fall
    shr al,4 
    add al,byte[carryToAdd]             ;adding the carry from the last time
    mov byte[ecx], al                   ;changing the data that we had in the link

    mov byte[carryToAdd],ah             ;saving the value in the memory to add it to the next link
    mov ecx,dword[ecx+5]

    cmp ecx,ebx
    jz .finished
    jmp .loopy


    .finished: 
    cmp byte[carryToAdd],0
    jz final

    mov al,byte[carryToAdd]
    call alocation
    mov ecx,dword[tmp]
    mov byte[ecx],al
    mov dword[ecx+5],ebx

    call moveCapacity
    mov edx,dword[myStack+ebx]
    mov dword[ecx+1],edx
    mov dword[edx+5],ecx
    mov dword[myStack+ebx],ecx

final:
  
    call moveCapacity
    mov eax,dword[myStack+ebx]              ;first link
    add ebx,4
    mov edx,dword[myStack+ebx]              ;last link


    dec dword[capacity]
    call moveCapacity
    mov ecx ,dword[myStack+ebx]          
    pushad
    call lineFree
    popad

    mov ecx,myStack
    add ecx,ebx
    mov dword[myStack+ebx] ,eax            ;first link
    mov dword[eax+5],ecx
    add ebx,4
    mov dword[myStack+ebx] ,edx            ;last link

    call moveCapacity
    mov ecx,dword[myStack+ebx]
    
    mov eax,myStack
    add eax,ebx

    .lastcheck:
    cmp byte[ecx],0
    jnz .con
    cmp dword[ecx+1],0
    jz .con
    mov edx,dword[ecx+1]
    mov dword[myStack+ebx],edx
    mov dword[edx+5],eax
    call freeing
    mov ecx,edx
    jmp .lastcheck


    .con:
    cmp byte[debug],1
    jz debugMode
	jmp input


pushNum:
    cmp byte[capacity],5                ;checking if the stack is full
    jz errorPrint2                      ;if yes printing error message
    
    xor eax,eax
    inc dword[capacity]
    call moveCapacity

    convert:
        cmp byte[edx],0                 ;checking if the end of the string
        jz lastC                        ;ready to next input
    
        cmp byte[edx],0x30
        jz .ziro

    .con:
        cmp byte[edx], 0x40             ;chacking if the next char bigger or even to A
        jg .upTOten
        jmp .lessThenTen
    
    .upTOten:                   
        mov al,byte[edx]                ;saving the next char value in eax
        sub eax,55                      ;taking the real hex value
        inc edx
        jmp myPush                      ;push it to the stack
        
        
    .lessThenTen:
        mov al,byte[edx]
        sub eax,48
        inc edx
        jmp myPush

     .ziro:
        cmp byte[first],1
        jz .returnn
        jmp .con

        .returnn:
        inc edx
        jmp convert
myPush:
        call alocation
        mov ecx, dword[tmp]
        
        mov byte[ecx], al               ;in cell 0 we save the hex value
        mov dword[ecx+1],0              ;in cell 1-4 we save the next link
        
        cmp byte[first],1               ;checking the flag first
        jz firstPush
        
        push ebx
        mov ebx,dword[linkkk]           ;saving in ebx the adress of prev link
        mov dword[ebx+1],ecx            ;saving the adress of curr as next in the prev link
        mov dword[ecx+5],ebx            ;in cell 5-8 we save the prev link
        pop ebx                         ;return ebx to capacity value

        mov dword[linkkk],ecx           ;saving the link
        jmp convert
        
firstPush:
        call moveCapacity
        mov dword[myStack+ebx],ecx      ;save the first link at stack+capacity                        
        mov byte[first],0               ;changing the flag
        mov eax,myStack
        add eax, ebx                    ;going to the right place in the stack to put the in the link the prev (the stack)
        mov dword[ecx+5],eax
        mov dword[linkkk],ecx           ;saving the link

        jmp convert

lastC:
    cmp byte[first],1                   ;checkking the flag
    jz .addZiro                         ;if we didnt add link yet the its ziro
    xor ecx,ecx
    mov ecx,dword[linkkk]               ;moving the last link to ecx
    call moveCapacity
    add ebx,4
    mov dword[myStack+ebx],ecx          ;saving the last link in the currect place in the stack

    cmp byte[debug],1
    jz debugMode
    jmp input

    .addZiro:
    call alocation

    mov ecx, dword[tmp]
    mov byte[ecx], 0                ;in cell 0 we save the hex value
    mov dword[ecx+1],0              ;in cell 1-4 we save the next link

    jmp firstPush


powerNeg:
    cmp byte[capacity],2
    jl errorPrint1

    inc dword[operetions]

    dec dword[capacity]
    call moveCapacity
    mov edx,dword[myStack+ebx]          ;first link of y

    inc dword[capacity]
    mov ecx,8
    .myLoop:                            ;counting num of links of y, we have only 2^32 address in our system
    shl eax,4
    add al,byte[edx]
    mov edx,dword[edx+1]
    cmp edx,0
    jz .con
    loop .myLoop,ecx

    jmp longer

    .con:
    xor edx,edx
    mov ebx,4
    div ebx
    mov byte[carryToAdd],dl

    call moveCapacity
    add ebx,4
    mov ecx,dword[myStack+ebx]          ;last link of x
    sub ebx,4
    add ebx,myStack

    .myloop2:
    cmp eax,0
    jz doCarry
    mov edx,dword[ecx+5]
    call freeing
    mov ecx,edx
    cmp ecx,ebx
    jz .longer
    dec eax
    jmp .myloop2

  .longer:
    dec dword[capacity]
    call moveCapacity
    mov ecx,dword[myStack+ebx]
    call lineFree
    jmp doziro

    doCarry:
    mov dword[ecx+1],0
    call moveCapacity
    add ebx,4
    mov dword[myStack+ebx],ecx
    sub ebx,4
    mov ebx,dword[myStack+ebx]

    xor ecx,ecx
    mov cl,byte[carryToAdd]
    cmp cl,0
    jz final
    mov ch,1
    shl ch,cl
    mov byte[carryToAdd],ch
    mov ch,0
  

.myloop:
        xor edx,edx
        xor eax,eax
        mov dl,byte[carryToAdd]
        mov al,byte[ebx]

        div dl                              ;al=num ah=reminder
        add al,ch
                                            ;founed in the net that the only reg that can do shift(not immediate) is cl
        mov ch,ah                           ;saving the reminder in ch
        ror ch,cl                           ;rotate the bits
        shr ch,4                            ;need hex number

        mov byte[ebx],al
        mov edx, dword[ebx+1]
        mov ebx,edx
        cmp ebx,0
        jz .final
        jmp .myloop

        .final:

        call moveCapacity
        mov ecx,dword[myStack+ebx]
        cmp byte[ecx],0
        jnz final

        mov eax,dword[ecx+1]
        mov edx,dword[ecx+5]
        call freeing
        mov dword[eax+5],edx
        mov dword[myStack+ebx],eax
        jmp final

    longer:
        call moveCapacity

        mov ecx,dword[myStack+ebx]
        call lineFree

        dec dword[capacity]
        call moveCapacity
        mov ecx,dword[myStack+ebx]
        call lineFree

        doziro:
        call alocation
        call moveCapacity

        mov ecx,dword[tmp]
        mov byte[ecx],0                     ;data
        mov dword[ecx+1],dword 0            ;next
        mov eax,myStack
        add eax,ebx
        mov dword[ecx+5],eax                ;prev

        mov dword[myStack+ebx],ecx          ;first link
        add ebx,4
        mov dword[myStack+ebx],ecx          ;last link


    cmp byte[debug],1
    jz debugMode
    jmp input

    
squareRoot:
xor edx,edx
call moveCapacity
mov ecx,dword[myStack+ebx]

inc dword[operetions]


.myLoop:
cmp ecx,0
jz .con
inc eax
mov ecx,dword[ecx+1]
jmp .myLoop

.con:
mov ebx,2
div ebx
cmp edx,1
jz .odd
jmp .even

.odd:
mov dword[rootCount],2
xor eax,eax
call moveCapacity
mov ecx,dword[myStack+ebx]

mov bl,byte[ecx]
mov edx,myTable
add edx,4
mov al,byte[edx]
mov byte[char],al

mul al
cmp bx,ax
jg .bigger
jl .smaller
jz .firstLink

.even:
mov dword[rootCount],4
xor eax,eax
call moveCapacity
mov ecx,dword[myStack+ebx]

xor ebx,ebx

mov bl,byte[ecx]
shl bl,4
mov ecx,dword[ecx+1]
add bl,byte[ecx]

mov edx,myTable
add edx,8
mov al,byte[edx]

mov byte[char],al
mul al
cmp bx,ax
jg .bigger
jl .smaller
jz .firstLink

.bigger:
cmp dword[rootCount],256
jz .firstLink
add edx,dword[rootCount]
push eax
push edx
mov dl,2
mov eax,dword[rootCount]
div dl
mov dword[rootCount],eax
pop edx
pop eax
mov al,byte[edx]
mov byte[char],al
mul al
cmp bx,ax
jg .bigger
jl .smaller
jz .firstLink


.smaller:
cmp dword[rootCount],256
jz .before
sub edx,dword[rootCount]
push eax
push edx
mov dl,2
mov eax,dword[rootCount]
div dl
mov dword[rootCount],eax
pop edx
pop eax
mov al,byte[edx]
mov byte[char],al
mul al
cmp bx,ax
jg .bigger
jl .smaller
jz .firstLink

.before:
dec edx
mov al,byte[edx]
mov byte[char],al
jmp .firstLink


.firstLink:
mov dword[rootCount],4
mov al,byte[char]
pushad
call alocation
mov ebx,dword[tmp]
mov byte[ebx],al                    ;the first value of the ans
mov dword[ebx+1],0                  ;next link
mov dword[ebx+5],mulStack           ;prev link
mov dword[mulStack],ebx             ;cell 0 holding the first link of ans
push eax
mov eax, 4
mov dword[mulStack+eax],ebx         ;last link 
pop eax


call alocation
mov ebx,dword[tmp]
add al,al
mov byte[ebx],al                    ;the first value of the num that changes
mov dword[ebx+1],0
mov dword[ebx+5],mulStack           

mov eax,8
mov dword[mulStack+eax],ebx         ;cell 8 holding the first link of the changing num
add eax,4
mov dword[mulStack+eax],ebx         ;last link
add eax,4
mov dword[mulStack+eax],1           ;we have one link in the list

popad
pushad
mul al
sub bl,al

call alocation
mov edx,dword[tmp]
mov byte[edx],bl                    ;this is the first value after sub
mov dword[edx+1],0                  ;next link
mov dword[edx+5],mulStack           ;prev link

mov eax,20
mov dword[mulStack+eax],edx         ;cell 20 holding the first link of the num that we taking from the origin
add eax,4
mov dword[mulStack+eax],edx         ;last linksquareRoot
add eax,4
mov dword[mulStack+eax],1           ;we have one link in the list

popad
mov ecx,dword[ecx+1]
mov dword[lastLink],ecx
jmp con

con:
mov ecx,dword[lastLink]
cmp ecx,0
jz finishedHere
mov dword[rootCount],4

call take2
mov dword[lastLink],ecx
call addlink
mov edx,myTable
add edx,8

cheking:
pushad
mov eax,16
mov ecx,dword[mulStack+eax]

add eax,12
mov ebx,dword[mulStack+eax]

cmp ebx,ecx
jnl doEverything

this:
mov eax,4
mov ecx,dword[mulStack+eax]
call alocation
mov edx,dword[tmp]
mov byte[edx],0
mov dword[edx+1],0
mov dword[edx+5],ecx
mov dword[ecx+1],edx
mov dword[mulStack+eax],edx

popad
jmp con

doEverything:
popad
call addNum
call dupli
call multi
call compare
mov eax,dword[forClean]
cmp eax,0
jz .founed
jl .less
jg .bigger

.less:
cmp dword[rootCount],256
jz .beforeFound
sub edx,dword[rootCount]
push eax
push edx
mov dl,2
mov eax,dword[rootCount]
div dl
mov dword[rootCount],eax
pop edx
pop eax
pushad
mov ecx,dword[link2]
call lineFree
jmp doEverything

.bigger:
cmp dword[rootCount],256
jz .founed
add edx,dword[rootCount]
push eax
push edx
mov dl,2
mov eax,dword[rootCount]
div dl
mov dword[rootCount],eax
pop edx
pop eax
pushad
mov ecx,dword[link2]
call lineFree
jmp doEverything

.beforeFound:
dec edx
mov ecx,dword[link2]
pushad
call lineFree
popad

call addNum
call dupli
call multi
jmp .founed

.founed:
call substract
mov ecx,dword[link2]
call lineFree

mov bl,byte[edx]
call alocation
mov ecx,dword[tmp]
mov byte[ecx],bl
mov dword[ecx+1],0

mov eax,4
mov edx,dword[mulStack+eax]
mov dword[edx+1],ecx
mov dword[ecx+5],edx
mov dword[mulStack+eax],ecx

mov eax,12
mov ecx,dword[mulStack+eax]
mov byte[ecx],bl
call addLast
jmp con


finishedHere:
call moveCapacity
mov ecx,dword[myStack+ebx]
pushad
call lineFree
popad
mov ecx,dword[mulStack]
mov eax,myStack
add eax,ebx
mov dword[ecx+5],eax
mov dword[myStack+ebx],ecx

mov eax,4
add ebx,4
mov ecx,dword[mulStack+eax]
mov dword[myStack+ebx],ecx

pushad
mov eax,8
mov ecx,dword[mulStack+eax]
call lineFree

mov eax,20
mov ecx,dword[mulStack+eax]
call lineFree
popad

    cmp byte[debug],1
    jz debugMode
    jmp input 


addLast:
pushad
mov al,byte[ecx]
shl al,4
shl bl,4
add al,bl
jnc .finished

.smallLoop:
shr al,4
mov byte[ecx],al
mov ecx,dword[ecx+5]
mov al,byte[ecx]
shl al,4
add al,16
jnc .finished
jmp .smallLoop

.finished:
shr al,4
mov byte[ecx],al
popad
ret

substract:
pushad
mov byte[carryToAdd],0
mov ecx,dword[link3]                    ;last link of the duplicated
mov eax,24
mov edx,dword[mulStack+eax]             ;last link of the origin

.myLoop:
mov bl,byte[edx]
sub bl,byte[ecx]
sub bl,byte[carryToAdd]
cmp bl,0
jnl .noC
add bl,16
mov byte[carryToAdd],1
jmp .con
.noC:
mov byte[carryToAdd],0
.con:
mov byte[edx],bl
mov edx,dword[edx+5]
mov ecx,dword[ecx+5]
cmp edx,mulStack
jz .finished
cmp ecx,link2
jz .finished

jmp .myLoop

.finished:
mov eax,20
mov ecx,dword[mulStack+eax]
add eax,8
.smallLoop:
cmp byte[ecx],0
jnz .over
dec dword[mulStack+eax]
mov edx,dword[ecx+1]
mov ebx,dword[ecx+5]
call freeing
mov ecx,edx
mov dword[ecx+5],ebx
jmp .smallLoop

.over:
sub eax,8
mov dword[mulStack+eax],ecx
popad
ret

dupli:
pushad
mov eax,16
mov edx,dword[mulStack+eax]
mov dword[tmpCounter],edx

mov eax,8
mov edx,dword[mulStack+eax]

call alocation
mov ecx,dword[tmp]
mov al,byte[edx]
mov byte[ecx],al

mov dword[ecx+1],0
mov dword[ecx+5],link2
mov dword[link2],ecx

mov edx,dword[edx+1]
cmp edx,0
jnz .myLoop
jmp .end

.myLoop:
mov ebx,ecx
call alocation
mov ecx,dword[tmp]
mov al,byte[edx]
mov byte[ecx],al

mov dword[ecx+1],0
mov dword[ecx+5],ebx
mov dword[ebx+1],ecx

mov dword[link3],ecx
mov edx,dword[edx+1]
cmp edx,0
jnz .myLoop

.end:
popad
ret

multi:
pushad
xor eax,eax
xor bh,bh
mov ecx,dword[link3]            ;last link of duplicated list

.myLoop:
mov al,byte[ecx]
shl al,4
mul bl
shr al,4
add al,bh
cmp al,15
jng .conn
add ah,1
sub al,16

.conn:
mov bh,ah
mov byte[ecx],al
mov ecx,dword[ecx+5]
cmp ecx,link2
jz .end
jmp .myLoop

.end:
cmp bh,0
jz .finished

call alocation
mov eax,dword[tmp]
mov ecx,dword[link2]

mov dword[link2],eax
mov byte[eax],bh
mov dword[eax+1],ecx
push edx
mov edx,dword[ecx+5]
mov dword[eax+5],edx
pop edx
mov dword[ecx+5],eax
inc dword[tmpCounter]

.finished:
popad
ret

compare:
pushad
mov eax,28
mov edx,dword[mulStack+eax]
cmp edx,dword[tmpCounter]
jl .smaller
jg .bigger

mov ecx,dword[link2]                    ;first link of multiplied num

mov eax,20
mov edx,dword[mulStack+eax]             ;first link of num that taken from origin

.myLoop:
cmp ecx,0
jz .same
xor eax,eax
mov al,byte[ecx]
mov ah,byte[edx]

cmp ah,al
jg .bigger
jl .smaller

mov ecx,dword[ecx+1]
mov edx,dword[edx+1]
jmp .myLoop

.bigger:
popad
mov dword[forClean],1
ret

.smaller:
popad
mov dword[forClean],-1
ret

.same:
popad
mov dword[forClean],0
ret

addNum:
xor ebx,ebx
mov bl,byte[edx]
mov eax,dword[linkkk]
mov byte[eax],bl
ret


addlink:
pushad
mov eax,12
mov edx,dword[mulStack+eax]

call alocation
mov ecx,dword[tmp]
mov byte[ecx],0
mov dword[ecx+1],0
mov dword[ecx+5],edx
mov dword[edx+1],ecx

mov dword[mulStack+eax],ecx
mov dword[linkkk],ecx
add eax,4
inc dword[mulStack+eax]
popad
ret


take2:

mov eax,24
mov ebx,dword[mulStack+eax]         ;last of the origin 
add eax,4
inc dword[mulStack+eax]
inc dword[mulStack+eax]

xor eax,eax
mov al,byte[ecx]

call alocation
mov edx,dword[tmp]
mov byte[edx],al
mov dword[edx+1],0
mov dword[edx+5],ebx
mov dword[ebx+1],edx

mov ecx,dword[ecx+1]
mov al,byte[ecx]
mov ebx,edx

call alocation
mov edx,dword[tmp]
mov byte[edx],al
mov dword[edx+1],0
mov dword[edx+5],ebx
mov dword[ebx+1],edx

mov eax,24
mov dword[mulStack+eax],edx
mov ecx,dword[ecx+1]


pushad
mov eax,20
mov ecx,dword[mulStack+eax]

.smallLoop:                         ;to delete the links with 0 in the biggening
mov ebx,dword[ecx+1]
cmp ebx,0
jz .con
cmp byte[ecx],0
jnz .con
call freeing
mov dword[mulStack+eax],ebx
mov dword[ebx+5],mulStack
add eax,8
dec dword[mulStack+eax]
sub eax,8
mov ecx,ebx
jmp .smallLoop

.con:
popad
ret



errorPrint1:
	push error1
	push format_string
	call printf
    push dword 0
    call fflush
	add esp, 12
    inc dword[operetions]
	jmp input
	
errorPrint2:
	push error2
	push format_string
	call printf
	push dword 0
    call fflush
	add esp, 12
	jmp input    

errorPrint3:
	push error3
	push format_string
	call printf
	push dword 0
    call fflush
	add esp, 12
    inc dword[operetions]
	jmp input  
	
