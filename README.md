# Movie-Library
Приложение для ведения личного списка фильмов. Позволяет добавлять записи, фильтровать их и автоматически сохраняет данные в JSON-файл.
import sys
import json
import os
from PyQt6.QtWidgets import (QApplication, QMainWindow, QWidget, QVBoxLayout, 
                             QHBoxLayout, QLabel, QLineEdit, QComboBox, 
                             QPushButton, QTableWidget, QTableWidgetItem, QMessageBox)

class MovieLibrary(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Movie Library")
        self.setGeometry(100, 100, 600, 450)
        self.data_file = "movies.json"
        self.movies = self.load_data()
        
        self.init_ui()
        self.update_table(self.movies)

    def init_ui(self):
        main_layout = QVBoxLayout()
        
        # --- Форма ввода ---
        form_layout = QHBoxLayout()
        self.input_title = QLineEdit(placeholderText="Название")
        self.input_genre = QLineEdit(placeholderText="Жанр")
        self.input_year = QLineEdit(placeholderText="Год")
        self.input_rating = QLineEdit(placeholderText="Рейтинг (0-10)")
        
        form_layout.addWidget(self.input_title)
        form_layout.addWidget(self.input_genre)
        form_layout.addWidget(self.input_year)
        form_layout.addWidget(self.input_rating)
        
        add_btn = QPushButton("Добавить фильм")
        add_btn.clicked.connect(self.add_movie)
        
        # --- Фильтрация ---
        filter_layout = QHBoxLayout()
        self.filter_genre = QLineEdit(placeholderText="Фильтр по жанру")
        self.filter_year = QLineEdit(placeholderText="Фильтр по году")
        filter_btn = QPushButton("Применить фильтр")
        filter_btn.clicked.connect(self.apply_filter)
        reset_btn = QPushButton("Сброс")
        reset_btn.clicked.connect(lambda: self.update_table(self.movies))

        filter_layout.addWidget(self.filter_genre)
        filter_layout.addWidget(self.filter_year)
        filter_layout.addWidget(filter_btn)
        filter_layout.addWidget(reset_btn)

        # --- Таблица ---
        self.table = QTableWidget(0, 4)
        self.table.setHorizontalHeaderLabels(["Название", "Жанр", "Год", "Рейтинг"])

        main_layout.addLayout(form_layout)
        main_layout.addWidget(add_btn)
        main_layout.addLayout(filter_layout)
        main_layout.addWidget(self.table)

        container = QWidget()
        container.setLayout(main_layout)
        self.setCentralWidget(container)

    def add_movie(self):
        title = self.input_title.text()
        genre = self.input_genre.text()
        year = self.input_year.text()
        rating = self.input_rating.text()

        # Валидация
        if not (title and genre and year and rating):
            return QMessageBox.warning(self, "Ошибка", "Заполните все поля!")
        
        if not year.isdigit():
            return QMessageBox.warning(self, "Ошибка", "Год должен быть числом!")
        
        try:
            r = float(rating)
            if not (0 <= r <= 10): raise ValueError
        except ValueError:
            return QMessageBox.warning(self, "Ошибка", "Рейтинг должен быть от 0 до 10!")

        movie = {"title": title, "genre": genre, "year": year, "rating": rating}
        self.movies.append(movie)
        self.save_data()
        self.update_table(self.movies)
        
        # Очистка полей
        self.input_title.clear()
        self.input_genre.clear()
        self.input_year.clear()
        self.input_rating.clear()

    def update_table(self, data):
        self.table.setRowCount(0)
        for row_data in data:
            row = self.table.rowCount()
            self.table.insertRow(row)
            self.table.setItem(row, 0, QTableWidgetItem(row_data["title"]))
            self.table.setItem(row, 1, QTableWidgetItem(row_data["genre"]))
            self.table.setItem(row, 2, QTableWidgetItem(row_data["year"]))
            self.table.setItem(row, 3, QTableWidgetItem(row_data["rating"]))

    def apply_filter(self):
        g_filter = self.filter_genre.text().lower()
        y_filter = self.filter_year.text()
        
        filtered = [m for m in self.movies if 
                    (g_filter in m["genre"].lower()) and 
                    (y_filter == "" or y_filter == m["year"])]
        self.update_table(filtered)

    def save_data(self):
        with open(self.data_file, "w", encoding="utf-8") as f:
            json.dump(self.movies, f, ensure_ascii=False, indent=4)

    def load_data(self):
        if os.path.exists(self.data_file):
            with open(self.data_file, "r", encoding="utf-8") as f:
                return json.load(f)
        return []

if __name__ == "__main__":
    app = QApplication(sys.argv)
    window = MovieLibrary()
    window.show()
    sys.exit(app.exec())

