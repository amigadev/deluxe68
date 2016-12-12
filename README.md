# Deluxe68

Deluxe68 is a simple (stupid) register allocator frontend for 68k assembly. It
is a source to source translator, so you'll need your regular assembler to
assemble its output. All is does it automate some tedious register allocation
for you.

## Usage

Usage is simple:

```
  deluxe68 input.s output.s
```

## Marking up source code

The following extensions are provided:

### Allocating registers

To pull from the data register pool, use `@dreg`:

```
		@dreg	a, b, [...]
		moveq	#0,@a
		moveq	#1,@b
```

Similarly, `@areg` allocates address registers. The stack pointer is
automatically reserved and will never be allocated:

```
		@areg	ptr
		lea	foo(pc),@ptr
```

### Using allocated registers

You can subsitute `@name` for a register in any instruction or macro
invokation. The only caveat is if the register has been spilled, in which case
you'll instead get a reference to the stack which can generate a
memory-to-memory instruction that doesn't assemble. In that case, rework the
code.

### Spilling and restoring registers

To explicitly spill a named register to the stack (returning it to the pool) you can use `@spill`:

```
		@dreg	a
		moveq	#0,@a
		@spill	a		; a is now on stack
		...
		...			; more code involving more data register allocation
		...
		@restore a		; a is now back in the same register it lived in before
```

`@spill` and `@restore` can also work with real registers, which is useful when
you want to call some external code using a particular calling interface:

```
		@spill	a0,d1
		move.l	@foo,a0
		move.l	@bar,d1
		bsr	SomeExternalCode
		@restore a0,d1
```

If `a0` or `d1` are not allocated, the spill does nothing.

### Procedures

Mark a procedure entry point with `@proc ProcedureName(<reg>: name, [<reg>: name ...])`.
This accomplishes two things:

- It generates an automatic `movem.l` that stores all touched registers to the stack
- All live registers are killed automatically

Similarly, instead of `rts`, use `@endproc`. This puts the inverse `movem.l` in
place, and also emits the `rts` instruction.

Any registers declared in the procedure header are automatically live and not
available for allocation in the procedure. You can however `@kill` them to
return them to the pool.

### License

This software is available under the BSD 2-clause license:

Copyright (c) 2016, Andreas Fredriksson
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice, this
   list of conditions and the following disclaimer.

2. Redistributions in binary form must reproduce the above copyright notice,
   this list of conditions and the following disclaimer in the documentation
   and/or other materials provided with the distribution.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

