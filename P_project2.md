## 파이썬 프로젝트

파이썬으로 크롤링을 사용하여 영화 순위 GUI 어플리케이션을 개발해보았다. 우선 프로젝트 실행 계획서를 작성해 보았다.

**프로젝트 보고서: 영화 순위 GUI 애플리케이션 개발**

---

**1. 개요**

이 보고서는 PyQt5와 웹 스크래핑 기술을 활용하여 개발된 영화 순위 GUI 애플리케이션에 대한 개요와 실행 결과를 기술한다.

---

**2. 목표**

이 프로젝트의 목표는 다음과 같다:

- PyQt5를 사용하여 사용자 친화적인 GUI를 구현한다.
- 웹 스크래핑을 통해 실시간 영화 순위 정보를 가져와 화면에 표시한다.
- 이미지 다운로드와 PyQt5의 테이블 위젯을 활용하여 영화 정보를 포함한 테이블을 구성한다.

---

**3. 기술적인 세부사항**

- 사용 언어: Python
- 라이브러리: PyQt5, requests, BeautifulSoup
- 개발 환경: Windows 10, PyCharm IDE

---

**4. 주요 기능**
- fetch_movie_data() 함수: 웹 페이지에서 영화 정보를 스크래핑하여 제목, 예매율, 개봉일, 포스터 이미지 경로를 추출한다.
- download_image(url) 함수: 주어진 이미지 URL에서 이미지를 다운로드하고 QPixmap 객체로 변환한다.
- MovieTable 클래스: PyQt5의 QTableWidget을 상속하여 영화 정보를 표시하는 테이블을 생성한다. / 영화 데이터를 받아서 테이블에 채우고, 포스터 이미지를 다운로드하여 각 행에 이미지를 표시한다.
- main() 함수: PyQt5 애플리케이션을 초기화하고 영화 데이터를 가져와 MovieTable을 생성하여 GUI를 구성한다. / 사용자에게 실시간 영화 순위 정보를 보여주는 GUI를 실행한다.

---

**5. 실행 및 테스트**

- Python 3.6 이상을 설치한다.
- 필요한 라이브러리를 설치한다 (requests, beautifulsoup4, PyQt5).
- 소스 코드를 실행하여 GUI 애플리케이션을 실행한다.
- GUI에서 실시간 영화 순위 정보를 확인한다.
- 이미지 다운로드 기능을 테스트하여 이미지가 정상적으로 표시되는지 확인한다.

---

**6. 결론**

이 프로젝트를 통해 PyQt5와 웹 스크래핑 기술을 활용하여 실시간 영화 순위를 보여주는 GUI 애플리케이션을 성공적으로 개발하였다. 사용자는 편리하게 영화 정보를 확인할 수 있고, 이미지 다운로드 기능도 정상적으로 작동하여 완성도 높은 애플리케이션이 되었다.

향후에는 네트워크 요청 시 timeout 설정 추가와 이미지 다운로드 실패에 대한 세부적인 처리를 통해 사용성과 안정성을 더욱 향상시킬 수 있을 것이다.

아래는 관련 코드이다.
```Python
import sys
import requests
from bs4 import BeautifulSoup
from PyQt5.QtWidgets import QApplication, QTableWidget, QTableWidgetItem, QLabel, QVBoxLayout, QWidget
from PyQt5.QtGui import QPixmap
from PyQt5.QtCore import Qt

# 사용자 에이전트 설정: 일부 웹사이트가 비브라우저 트래픽을 차단하는 것을 방지하기 위해
HEADERS = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.3'
}

# 영화 데이터를 가져오는 함수
def fetch_movie_data():
    # 실시간 영화 순위 페이지 URL
    URL = "https://m.moviechart.co.kr/rank/realtime/index/image"
    # 페이지 요청 및 응답 저장
    page = requests.get(URL, headers=HEADERS)
    # BeautifulSoup 객체 생성하여 HTML 파싱
    soup = BeautifulSoup(page.content, 'html.parser')
    # 영화 목록 아이템을 찾음
    movie_items = soup.find_all('li', class_='movieBox-item')[:20]
    movie_data = []

    # 영화 목록 아이템을 순회하며 필요한 정보 추출
    for index, item in enumerate(movie_items, start=1):
        title = item.find('h3').text.strip()
        rate = item.find('li', class_='ticketing').find('span').text.strip()
        release_date = item.find('li', class_='movie-launch').text.strip().replace('개봉일 ', '')
        img_path = item.find('img')['src'].split('source=')[1]
        full_img_path = f"https:{img_path}" if not img_path.startswith('http') else img_path
        movie_data.append((index, title, rate, release_date, full_img_path))

    return movie_data

# 이미지를 다운로드하는 함수
def download_image(url):
    try:
        response = requests.get(url, headers=HEADERS)
        response.raise_for_status()
        pixmap = QPixmap()
        pixmap.loadFromData(response.content)
        return pixmap
    except Exception as e:
        print(f"이미지 다운로드 중 오류 발생: {e}")
    return QPixmap()

# 영화 테이블을 표시하는 클래스
class MovieTable(QTableWidget):
    def __init__(self, data):
        super().__init__()
        self.setRowCount(len(data))
        self.setColumnCount(5)
        self.setHorizontalHeaderLabels(['순위', '포스터', '제목', '예매율', '개봉일'])
        self.set_data(data)
        self.resizeColumnsToContents()

        # 행 번호(왼쪽에 있는 인덱스 숫자)를 숨깁니다.
        self.verticalHeader().setVisible(False)

    # 테이블 데이터 설정 함수
    def set_data(self, data):
        for row, (index, title, rate, release_date, img_path) in enumerate(data):
            self.setItem(row, 0, QTableWidgetItem(str(index)))
            self.setItem(row, 2, QTableWidgetItem(title))
            self.setItem(row, 3, QTableWidgetItem(rate))
            self.setItem(row, 4, QTableWidgetItem(release_date))

            pixmap = download_image(img_path)
            if not pixmap.isNull():
                label = QLabel()
                label.setPixmap(pixmap.scaled(50, 75, Qt.KeepAspectRatio))
                self.setCellWidget(row, 1, label)


def main():
    app = QApplication(sys.argv)
    movie_data = fetch_movie_data()
    table = MovieTable(movie_data)
    layout = QVBoxLayout()
    layout.addWidget(table)

    window = QWidget()
    window.setLayout(layout)
    window.setWindowTitle("실시간 영화 순위")
    window.resize(605, 975)  # 창의 크기를 800x600으로 설정
    window.show()

    sys.exit(app.exec_())


if __name__ == "__main__":
    main()
```
