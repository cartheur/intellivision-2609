.PHONY: manual ocr clean

manual:
	latexmk -pdf KeyboardComponentServiceManual.tex

ocr:
	latexmk -pdf KeyboardComponentServiceManual-OCR.tex

clean:
	latexmk -c KeyboardComponentServiceManual.tex
	latexmk -c KeyboardComponentServiceManual-OCR.tex
