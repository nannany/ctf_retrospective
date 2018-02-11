# divN
>$ cat foo.c
>long long div(long long x) {
>    return x / N;
>}
>$ gcc -DN=$N -c -O2 foo.c
>$ objdump -d foo.o
>
>foo.o:     file format elf64-x86-64
>
>
>Disassembly of section .text:
>
>0000000000000000 
>:
>   0:	48 89 f8             	mov    %rdi,%rax
>   3:	48 ba 01 0d 1a 82 9a 	movabs $0x49ea309a821a0d01,%rdx
>   a:	30 ea 49 
>   d:	48 c1 ff 3f          	sar    $0x3f,%rdi
>  11:	48 f7 ea             	imul   %rdx
>  14:	48 c1 fa 30          	sar    $0x30,%rdx
>  18:	48 89 d0             	mov    %rdx,%rax
>  1b:	48 29 f8             	sub    %rdi,%rax
>  1e:	c3                   	retq   
>$ echo “HarekazeCTF{$N}” > /dev/null
>(Rev, 100 points)

foo.c を N=$N と定義したうえで、最適化してコンパイルした(-O2)もののアセンブリを読み解いて、$Nの値を探る問題。

アセンブラの記法は、%があることから判断してAT&T。つまり、movだったら左から右に移動している。

`mov    %rdi,%rax` で、引数としてわたってきた %rdi が %rax にコピーされる。
`movabs $0x49ea309a821a0d01,%rdx` で、%rdx(データレジスタ)に ‭5326122949484875009‬ が入る。
`sar    $0x3f,%rdi` で、%rdiには最上位1bitが最下位1bitとなり、それ以外はもともとの最上位1bitと同じになる。
`imul   %rdx` で、%raxの値（つまり、引数で渡されたx）と%rdxの値を乗算したものの、上位を%rdxに、下位を%raxに格納する。
`sar    $0x30,%rdx` で、%rdxの上位16bitが下位16bitとなり、上位48bitは最上位のbitと同じになる。
`mov    %rdx,%rax` で、%rdxを%raxにコピーする。
`sub    %rdi,%rax` で、%raxから%rdiを引いた値を%raxに格納する。

‭Nを求めるには、
‭(2**112) // ‭5326122949484875009‬ +1
をする。
（+1がなんで必要か正直分かってない。。）

* 関数の戻り値は rax で戻す。
* 引数はrdi, rsi, rdx, rcx, r8, r9の順でセットされる。（http://inaz2.hatenablog.com/entry/2014/07/04/001851）
* long long型は64bit以上の整数を表現することができる。