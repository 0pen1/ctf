## Can you read Pacifico? (misc/ppc, 400+1p)

### PL Version
[ENG](#eng-version)

Zadanie polega�o na napisaniu �amacza captchy. Captche mia�y nast�puj�cy format:

![](./captcha.png)

Nale�a�o rozwi�za� 1337 kod�w pod rz�d bezb��dnie w celu uzyskania flagi.

Jak nie trudno zauwa�y� konieczne b�dzie przetworzenie obrazu do wersji bardziej przyst�pnej do automatycznej analizy. Stosujemy algorytm, kt�ry przetworzy podany obraz do postaci czarnego tekstu na bia�ym tle. Wykonujemy to za pomoc� zestawu 2 funkcji:

	def fix_colors(im):
		colors_distribution = im.getcolors(1000)
		ordered = sorted(colors_distribution, key=lambda x: x[0], reverse=True)
		best_colors = [color[1] for color in ordered]
		if (255, 255, 255) in best_colors:
			best_colors.remove((255, 255, 255))
		if (0, 0, 0) in best_colors:
			best_colors.remove((0, 0, 0))
		best_colors = best_colors[:2]
		pixels = im.load()
		for i in range(im.size[0]):
			for j in range(im.size[1]):
				color = pixels[i, j]
				if color not in best_colors:
					pixels[i, j] = best_colors[0]
		return best_colors[0]


	def black_and_white(im, filling):
		black = (0, 0, 0)
		white = (255, 255, 255)
		pixels = im.load()
		for i in range(im.size[0]):
			for j in range(im.size[1]):
				color = pixels[i, j]
				if color == filling:
					pixels[i, j] = white
				else:
					pixels[i, j] = black
					
Pierwsza s�u�y do usuni�ciu z obrazu wszystkich opr�cz 2 dominujacych kolor�w (kt�rymi jest wype�nienie i tekst), przy czym szukajac dominuj�cych kolor�wnie bierzemy pod uwag� bia�ego i czarnego, aby nie wybra� kt�rego� z nich jako koloru wype�nienia.

Druga funkcja zamienia kolor tekstu na czarny a kolor wype�nienia na bia�y. W efekcie uzyskujemy:

![](./captcha_color.png)

A nast�pnie:

![](./captcha_bw.png)

Niestety jak nie trudno zauwa�y� nasze captche s� bardzo bardzo ma�e. Za ma�e �eby OCR by� w stanie poprawnie i bezb��dnie je odczyta�. Stosujemy wi�c inn� metod�. Jeden z naszych koleg�w po�wieci� si� i zmapowa� stosowan� czcionk� tworz�c pliki z ka�dym potrzebnym symbolem:

![](./alphabet.png)

Algorytm dekodowania captcha wygl�da� nast�pujaco:

1. Pobieramy captche
2. Poprawiamy kolory i zamieniamy na czarno-bia�e
3. Pobieramy prostok�t obejmujacy captch�
4. Dla ka�dego symbolu z alfabetu znajdujemy najlepsze dopasowanie go do captchy (takie gdzie najbardziej si� z ni� pokrywa), zapami�tujemy te� pozycje tego dopasowania
5. Wybieramy najlepiej dopasowany symbol
6. Usuwamy z captchy dopasowany symbol poprzez zamalowanie go na bia�o
7. Kroki 4-6 powtarzamy 6 razy, bo szukamy 6 symboli.
8. Sortujemy uzyskane symbole wzgl�dem pozycji dopasowania w kolejnosci rosnacej aby odtworzy� poprawn� kolejno�� symboli

Dla prezentowanej wy�ej captchy sesja dekodowania wygl�da tak (najlepiej dopasowany symbol oraz obraz po jego usuni�ciu):

`('H', 155)`

![](./h.png)

`('G', 122)`

![](./g.png)

`('U', 103)`

![](./u.png)

`('E', 99)`

![](./e.png)

`('Z', 86)`

![](./z.png)

`('9', 46)`

![](./9.png)

W wyniku czego uzyskujemy kod: `EUZGH9`

Dwie funkcje, o kt�rych warto wspomnie� to funkcja licz�ca jak dobre dopasowanie znale�li�my oraz funkcja usuwaj�ca symbole z obrazu.

	def test_configuration(im_pixels, start_x, start_y, symbol_len, symbol_h, symbol_pixels, symbol_x_min, symbol_y_min):
		counter = 0
		black = (0, 0, 0)
		for i in range(symbol_len):
			for j in range(symbol_h):
				if im_pixels[start_x + i, start_y + j] == black:
					if im_pixels[start_x + i, start_y + j] == symbol_pixels[symbol_x_min + i, symbol_y_min + j]:
						counter += 1
					else:
						counter -= 1
				elif symbol_pixels[symbol_x_min + i, symbol_y_min + j] == black:
					counter -= 1
		return counter
	
Dla zadanego offsetu (w poziomie - start_x oraz w pionie - start_y) na obrazie z captch� iterujemy po pikselach i por�wnujemy je z pikselami testowanego symbolu. Je�li trafimy na czarne piksele na obrazie to sprawdzamy czy wyst�puj� tak�e w symbolu i dodajemy lub odejmujemy 1 od licznika pasuj�cych pikseli. Je�li na obrazie mamy kolor bia�y a symbol ma czarne piksele to odejmujemy 1 od licznika. Ta druga cz�� jest do�� istotna, poniewa� w innym wypadku litery "ca�kowicie obejmuj�ce inne" mog�yby by� wskazane jako poprawne dopasowanie.

	def is_to_remove(symbol_pixels, x, y):
		black = (0, 0, 0)
		result = False
		for i in range(-1, 1):
			for j in range(-1, 1):
				result = result or symbol_pixels[x + i, y + j] == black
		return result


	def remove_used(picture_pixels, symbol, offset_x, offset_y, symbol_len, symbol_h):
		white = (255, 255, 255)
		symbol_x_min, _, symbol_y_min, _ = get_coords(symbol)
		symbol_pixels = symbol.load()
		for i in range(offset_x, offset_x + symbol_len + 1):
			for j in range(offset_y, offset_y + symbol_h + 1):
				if is_to_remove(symbol_pixels, symbol_x_min + i - offset_x, symbol_y_min + j - offset_y):
					picture_pixels[i, j] = white

Usuwaj�c symbol z obrazu iterujemy po obrazie zgodnie ze znalezionym offsetem i kolorujemy na bia�o te piksele, kt�re s� czarne tak�e w symbolu. Nie kolorujemy ca�ego boxa na bia�o! Jest to do�� istotne, bo captche czasem by�y takie:

![](./captcha4.png)

Jak wida� symbol `5` znajduje sie wewn�trz boxa obejmuj�cego liter� `T` i zamalowanie ca�ego boxa na bia�o "zniszczy" symbol `5`.

Kod ca�ego solvera dost�pny jest [tutaj](./captcha.py). Tak przygotowany solver + pypy i szybki internet pozwoli� po pewnym czasie uzyska� flag�:

`DCTF{6b91e112ee0332616a5fe6cc321e48f1}`
		
### ENG Version

The task was to create a captcha breaker. The captcha codes looked like this:

![](./captcha.png)

We had to solve 1337 examples in a row without any mistakes to get the flag.

As can be noticed we had to process the image to make it more useful for automatic analysis. We used an algorithm to get a black & white clean version of the text. We used 2 functions for this:

	def fix_colors(im):
		colors_distribution = im.getcolors(1000)
		ordered = sorted(colors_distribution, key=lambda x: x[0], reverse=True)
		best_colors = [color[1] for color in ordered]
		if (255, 255, 255) in best_colors:
			best_colors.remove((255, 255, 255))
		if (0, 0, 0) in best_colors:
			best_colors.remove((0, 0, 0))
		best_colors = best_colors[:2]
		pixels = im.load()
		for i in range(im.size[0]):
			for j in range(im.size[1]):
				color = pixels[i, j]
				if color not in best_colors:
					pixels[i, j] = best_colors[0]
		return best_colors[0]


	def black_and_white(im, filling):
		black = (0, 0, 0)
		white = (255, 255, 255)
		pixels = im.load()
		for i in range(im.size[0]):
			for j in range(im.size[1]):
				color = pixels[i, j]
				if color == filling:
					pixels[i, j] = white
				else:
					pixels[i, j] = black
					
First one removed all colors except for 2 dominant ones (filling and text), however we skip black and white when looking for dominants, so we don't pick one as filling.

Second function changes the text color to black and filling color to white. This was we get:

![](./captcha_color.png)

And then:

![](./captcha_bw.png)

Unfortunately, as you can notice the captchas are really small. Too small for OCR to give good results. Therefore, we proceed with a different method. One of our teammates mapped the whole alphabet of necessary symbols:

![](./alphabet.png)

Captcha-solving algorithm:

1. Download captcha
2. Fix colors and make black & white
3. Calculate a box with captcha letters
4. For every alphabet character we calculate the best positioning on the captcha (where it overlaps the most), we save also the offset of this positioning 
5. We pick the character with highest score
6. We remove the character from captcha by painting it white.
7. We perform steps 4-6 six times, because we are looking for 6 characters.
8. We sort the characters by the x_offset of the positioning of the character, to reconstruct the inital order of characters

For the captcha presented above the decoding session looks like this (the best matched character and picture after removal):

`('H', 155)`

![](./h.png)

`('G', 122)`

![](./g.png)

`('U', 103)`

![](./u.png)

`('E', 99)`

![](./e.png)

`('Z', 86)`

![](./z.png)

`('9', 46)`

![](./9.png)

As a result we get the code: `EUZGH9`

Two functions that are worth mentioning are the function to calculate matching score and function for removing the characters from picture.

	def test_configuration(im_pixels, start_x, start_y, symbol_len, symbol_h, symbol_pixels, symbol_x_min, symbol_y_min):
		counter = 0
		black = (0, 0, 0)
		for i in range(symbol_len):
			for j in range(symbol_h):
				if im_pixels[start_x + i, start_y + j] == black:
					if im_pixels[start_x + i, start_y + j] == symbol_pixels[symbol_x_min + i, symbol_y_min + j]:
						counter += 1
					else:
						counter -= 1
				elif symbol_pixels[symbol_x_min + i, symbol_y_min + j] == black:
					counter -= 1
		return counter

For given offset (horizontal - start_x and vertical - start_y) on the captcha picture we iterate over pixels and compare with symbol pixels. If we have black pixels on the picture we check if they are black also in symbol and we add or subtract 1 from the matched pixels counter. If on the picture we have white color but black in symbol we subtract 1 from counter. The second part os really important, because otherwise the large letters that "cover other letters entirely" could be selected as best match.

	def is_to_remove(symbol_pixels, x, y):
		black = (0, 0, 0)
		result = False
		for i in range(-1, 1):
			for j in range(-1, 1):
				result = result or symbol_pixels[x + i, y + j] == black
		return result


	def remove_used(picture_pixels, symbol, offset_x, offset_y, symbol_len, symbol_h):
		white = (255, 255, 255)
		symbol_x_min, _, symbol_y_min, _ = get_coords(symbol)
		symbol_pixels = symbol.load()
		for i in range(offset_x, offset_x + symbol_len + 1):
			for j in range(offset_y, offset_y + symbol_h + 1):
				if is_to_remove(symbol_pixels, symbol_x_min + i - offset_x, symbol_y_min + j - offset_y):
					picture_pixels[i, j] = white

In order to remove the symbol from picture we iterate over it starting from appropriate offset and color pixels white if they are black in the symbol. We don't color the whole box white! It is important because some captchas were like this:

![](./captcha4.png)

As can be seen the character `5` is inside the box surrounding letter `T` and paiting it white would "destroy" the character `5`. 

Code of the solver is available [here](./captcha.py).  This solver + pypy + fast internet connection resulted in:

`DCTF{6b91e112ee0332616a5fe6cc321e48f1}`