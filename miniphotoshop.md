## 1. 개요
이 프로그램은 넘파이와 PIL(Python Imaging Library)을 활용하여 이미지 처리를 위한 간단한 미니포토샵을 구현한 프로그램이다. Tkinter를 사용하여 GUI를 구성하고, 사용자가 이미지를 열고 저장하며 다양한 이미지 처리 기능을 수행할 수 있다.

---

## 2. 목표

- 이미지를 열고 저장할 수 있는 기능 제공
- 이미지를 반전시키고, 밝기를 조절하며, 회색조로 변환하는 등의 이미지 처리 기능 제공
- 이미지를 확대/축소하고, 상하 또는 좌우로 반전시키며, 회전하는 등의 변형 기능 제공

---

## 3. 기술적인 세부사항

- 프로그래밍 언어: Python
- 사용된 라이브러리: tkinter, PIL(Python Imaging Library), numpy
- 개발 환경: Visual Studio Code, Python 3.12
- 실행 환경: Windows 10

---

## 4. 주요 기능

### 이미지 열기와 저장

- 파일 메뉴에서 '파일 열기'를 선택하면 파일 다이얼로그가 열려 사용자가 이미지를 선택할 수 있습니다.
- '파일 저장'을 선택하면 현재 작업 중인 이미지를 사용자가 지정한 경로에 저장합니다.
  
### 이미지 처리

- 반전 이미지: 이미지의 색상을 반전시킵니다.
- 밝기 조절: 사용자가 입력한 값에 따라 이미지의 밝기를 조절합니다.
- 회색조 이미지: RGB 값을 평균내어 회색조 이미지로 변환합니다.
- 흑백 이미지: 임계값을 기준으로 흑백 이미지로 변환합니다.

### 이미지 변형

- 이미지 확대/축소: 사용자가 입력한 배율에 따라 이미지를 확대 또는 축소합니다.
- 이미지 반전: 이미지를 상하 또는 좌우로 반전시킵니다.
- 이미지 회전: 사용자가 선택한 각도에 따라 이미지를 회전시킵니다.

---

## 5. 실행 및 테스트

- 프로그램을 실행하고 '파일 열기'를 통해 이미지를 불러온 후, 각 기능을 테스트합니다.
- 이미지 처리, 변형 기능을 테스트하여 정상적으로 작동하는지 확인합니다.
- 다양한 이미지에 대해 테스트하여 예외 상황에 대비합니다.

---

## 6. 결론
이 프로그램은 간단한 이미지 처리 및 변형 기능을 제공하여 사용자가 이미지를 다양하게 가공할 수 있도록 도와줍니다. 사용자 편의성을 높이기 위해 GUI를 사용하고, 각 기능은 사용자 입력에 따라 동적으로 처리됩니다. 추가적인 기능이나 사용성 개선을 위해 피드백을 받고 지속적인 업데이트를 진행할 수 있습니다.

---

```Pthon
from tkinter import *
from tkinter.filedialog import *
from tkinter.simpledialog import *
from PIL import Image
import numpy as np

##함수 선언 부분##
def display(): 
    global window, canvas, paper, filename, inImage, outImage, inW, inH, outW, outH

    if canvas != None:
        canvas.destroy()

    window.geometry(str(outW) + 'x' + str(outH))
    canvas = Canvas(window, width=outW, height=outW)
    paper = PhotoImage(width=outW, height=outH)
    canvas.create_image((outW / 2, outH / 2), image = paper, state='normal')

    rgbString = ""
    for i in range(0, outH):
        tmpString = ""
        for k in range(0, outW):
            dataR = outImage[0][i][k]
            dataG = outImage[1][i][k]
            dataB = outImage[2][i][k]
            tmpString += "#%02x%02x%02x " % (dataR, dataG, dataB) # x 뒤에 한 칸 공백
        rgbString += "{" + tmpString + "} "                       # } 뒤에 한 칸 공백
    paper.put(rgbString)
    canvas.pack()

def func_open() : 
    global window, canvas, paper, filename, inImage, outImage, inW, inH, outW, outH
    filename = askopenfilename(parent=window, filetypes=(("Color 파일", \
                               "*.jpg;*.png;*.bmp;*.tif;"), ("모든파일", "*.*")))

    photo = Image.open(filename)  # PIL 객체
    inW = photo.width
    inH = photo.height
    inImage = np.empty((3, inH, inW), dtype=np.uint8)
    photoRGB = photo.convert('RGB')
    for i in range(inH):
        for k in range(inW):
            r, g, b = photoRGB.getpixel((k, i))
            inImage[0][i][k] = r
            inImage[1][i][k] = g
            inImage[2][i][k] = b

    outW = inW
    outH = inH
    outImage = inImage.copy()
    display()


def func_save() : 
    global window, canvas, paper, filename, inImage, outImage, inW, inH, outW, outH
    saveFp = asksaveasfile(parent=window, mode='wb', defaultextension='*.jpg',
                           filetypes=(("JPG 파일", "*.jpg"), ("모든 파일", "*.*")))
    if saveFp == '' or saveFp == None:
        return
    photoRGB = Image.new('RGB', (outW, outH))
    for i in range(outH):
        for k in range(outW):
            r, g, b = outImage[0][i][k], outImage[1][i][k], outImage[2][i][k]
            photoRGB.putpixel( (k,i), (r, g, b, 255))
    photoRGB.save(saveFp.name)


def func_exit():
    exit()

def func_reverse() :
    global window, canvas, paper, filename, inImage, outImage, inW, inH, outW, outH
    outW = inW
    outH = inH
    outImage = 255 - inImage
    display()


def func_bright() :
    global window, canvas, paper, filename, inImage, outImage, inH, inW, outH, outW
    outH = inH
    outW = inW
    value = askinteger("밝게", "값-->", minvalue=-1, maxvalue=255)
    inImage = inImage.astype(np.int16)
    outImage = inImage + value
    outImage = np.where(outImage > 255, 255, outImage)
    outImage = outImage.astype(np.uint8)
    inImage = inImage.astype(np.uint8)
    display()


def func_dark() :
    global window, canvas, paper, filename, inImage, outImage, inH, inW, outH, outW
    outH = inH
    outW = inW
    value = askinteger("어둡게", "값-->", minvalue=-1, maxvalue=255)
    inImage = inImage.astype(np.int16)
    outImage = inImage - value
    outImage = np.where(outImage < 0, 0, outImage)
    outImage = outImage.astype(np.uint8)
    inImage = inImage.astype(np.uint8)
    display()


def func_gray() : 
    global window, canvas, paper, filename, inImage, outImage, inH, inW, outH, outW
    ## 중요! 출력 영상 크기 결정 ##
    outH = inH
    outW = inW
    inImage = inImage.astype(np.int16)
    grayImage = (inImage[0] + inImage[1] + inImage[2]) / 3    # 2차원 배열
    grayImage = np.concatenate((grayImage, grayImage, grayImage))
    outImage = np.reshape(grayImage, (3,outH, outW))
    outImage = outImage.astype(np.uint8)
    inImage = inImage.astype(np.uint8)
    display()

def func_bw1(): 
    global window, canvas, paper, filename, inImage, outImage, inW, inH, outW, outH
    outW = inW
    outH = inH

    inImage = inImage.astype(np.int16)
    grayImage = (inImage[0] + inImage[1] + inImage[2]) / 3 # 2차원 배열
    grayImage = np.concatenate((grayImage, grayImage, grayImage))
    outImage = np.reshape(grayImage, (3, outH, outW))
    outImage = np.where(outImage < 128, 0, 255)
    outImage = outImage.astype(np.uint8)
    inImage = inImage.astype(np.uint8)
    display()


def func_bw2() : 
    global window, canvas, paper, filename, inImage, outImage, inW, inH, outW, outH
    outW = inW
    outH = inH
    inImage = inImage.astype(np.int16)
    grayImage = (inImage[0] + inImage[1] + inImage[2]) / 3 # 2차원 배열
    grayImage = np.concatenate((grayImage, grayImage, grayImage))
    outImage = np.reshape(grayImage, (3, outH, outW))
    value = np.mean(outImage)
    outImage = np.where(outImage < value, 0, 255)
    outImage = outImage.astype(np.uint8)
    inImage = inImage.astype(np.uint8)
    display() 


def func_mirror1() :
    global window, canvas, paper, filename, inImage, outImage, inW, inH, outW, outH
    outW = inW
    outH = inH
    outImage = np.flip(inImage,axis=1)
    display()

def func_mirror2() :
    global window, canvas, paper, filename, inImage, outImage, inW, inH, outW, outH
    outW = inW
    outH = inH
    outImage = np.flip(inImage, axis=2)
    display()


def func_rotate(times) :
    global window, canvas, paper, filename, inImage, outImage, inW, inH, outW, outH
    outW = inW
    outH = inH
    outImage = np.rot90(inImage, k=times, axes=(1,2))
    for i in range(times) :
        outW, outH = outH, outW
    display()


def func_zoomin() :
    global window, canvas, paper, filename, inImage, outImage, inW, inH, outW, outH
    scale = askinteger("확대", "배율-->", minvalue=-2, maxvalue=8)
    outW = inW * scale
    outH = inH * scale
    outImage = np.empty((3,outH, outW), dtype=np.uint8)
    for rgb in range(3) :
        for i in range(outH) :
            for k in range(outW) :
                outImage[rgb][i][k] = inImage[rgb][int(i/scale)][int(k/scale)]
    display()


def func_zoomout() :
    global window, canvas, paper, filename, inImage, outImage, inW, inH, outW, outH
    scale = askinteger("축소", "배율-->", minvalue=-2, maxvalue=8)
    outW = int(inW / scale)
    outH = int(inH / scale)
    outImage = np.empty((3, outH, outW), dtype=np.uint8)
    for rgb in range(3):
        for i in range(outH):
            for k in range(outW):
                outImage[rgb][i][k] = inImage[rgb][int(i * scale)][int(k * scale)]
    display()


##전역 변수 선언 부분##
window, canvas, paper, filename = [None]*4
inImage, outImage = [], []
inW, inH, outW, outH = [0]*4

##메인 코드 부분##
window = Tk()
window.geometry("500x500")
window.title("넘파이 미니 포토샵")

mainMenu = Menu(window)
window.config(menu = mainMenu)

fileMenu = Menu(mainMenu)
mainMenu.add_cascade(label = "파일", menu = fileMenu)
fileMenu.add_command(label="파일 열기", command = func_open)
fileMenu.add_command(label="파일 저장", command = func_save)
fileMenu.add_separator()
fileMenu.add_command(label="프로그램 종료", command = func_exit)

image1Menu = Menu(mainMenu)
mainMenu.add_cascade(label = "영상 처리(1)", menu = image1Menu)
image1Menu.add_command(label="반전이미지", command = func_reverse)
image1Menu.add_separator()
image1Menu.add_command(label="밝게", command = func_bright)
image1Menu.add_command(label="어둡게", command = func_dark)
image1Menu.add_separator()
image1Menu.add_command(label="회색이미지", command = func_gray)
image1Menu.add_command(label="흑백이미지(127 기준)", command=func_bw1)
image1Menu.add_command(label="흑백이미지(평균값)", command=func_bw2)

image2Menu = Menu(mainMenu)
mainMenu.add_cascade(label="영상 처리(2)", menu = image2Menu)
image2Menu.add_command(label="상하 반전", command = func_mirror1)
image2Menu.add_command(label="좌우 반전", command = func_mirror2)
image2Menu.add_separator()
image2Menu.add_command(label = "90도 회전", command = lambda:func_rotate(1))
image2Menu.add_command(label = "180도 회전", command = lambda:func_rotate(2))
image2Menu.add_command(label = "270도 회전", command = lambda:func_rotate(3))
image2Menu.add_separator()
image2Menu.add_command(label = "확대", command = lambda:func_zoomin)
image2Menu.add_command(label = "축소", command = lambda:func_zoomout)

window.mainloop()
```
<img width="437" alt="스크린샷 2024-04-16 155121" src="https://github.com/dawoon1229/Python_project/assets/164113758/ce6cca29-9e31-4ba6-807e-79558cf508dd">

원본

<img width="440" alt="반전이미지" src="https://github.com/dawoon1229/Python_project/assets/164113758/9068ab0e-9bf1-432d-bb04-a9dfa0a768b6">

반전이미지

<img width="438" alt="상하반전" src="https://github.com/dawoon1229/Python_project/assets/164113758/528d1301-9875-4f40-83b2-da4bb54d8c9d">

상하반전

<img width="749" alt="270도회전" src="https://github.com/dawoon1229/Python_project/assets/164113758/c9b90d7e-4b20-4cc2-988a-ad310d562fb6">

270도 회전





