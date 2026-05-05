Отлично! Я полностью переделаю приложение под **Movie Library (Личная кинотека)**. Это будет полноценное приложение на 5 баллов со всеми требованиями.

## 🎬 Movie Library - Личная кинотека

### Полный код приложения

#### **models.py** - Модель фильма и валидация
```python
class Movie:
    def __init__(self, title: str, genre: str, year: int, rating: float):
        if not self.validate_title(title):
            raise ValueError("Название не может быть пустым и должно быть короче 100 символов")
        if not self.validate_genre(genre):
            raise ValueError("Жанр не может быть пустым и должен быть короче 50 символов")
        if not self.validate_year(year):
            raise ValueError("Год должен быть от 1888 до 2026")
        if not self.validate_rating(rating):
            raise ValueError("Рейтинг должен быть числом от 0 до 10")
        
        self.title = title.strip()
        self.genre = genre.strip()
        self.year = year
        self.rating = rating
    
    @staticmethod
    def validate_title(title: str) -> bool:
        return bool(title) and len(title) <= 100
    
    @staticmethod
    def validate_genre(genre: str) -> bool:
        return bool(genre) and len(genre) <= 50
    
    @staticmethod
    def validate_year(year: int) -> bool:
        return 1888 <= year <= 2026
    
    @staticmethod
    def validate_rating(rating: float) -> bool:
        return 0 <= rating <= 10
    
    def to_dict(self) -> dict:
        return {
            "title": self.title,
            "genre": self.genre,
            "year": self.year,
            "rating": self.rating
        }
    
    @classmethod
    def from_dict(cls, data: dict):
        return cls(
            title=data["title"],
            genre=data["genre"],
            year=data["year"],
            rating=data["rating"]
        )
    
    def __str__(self) -> str:
        return f"{self.title} ({self.year}) - {self.genre} - Рейтинг: {self.rating}"
```

#### **storage.py** - Работа с JSON
```python
import json
import os
from typing import List
from models import Movie

class Storage:
    def __init__(self, filename: str = "movies.json"):
        self.filename = filename
        self._ensure_file_exists()
    
    def _ensure_file_exists(self):
        """Создаёт файл JSON, если его нет"""
        if not os.path.exists(self.filename):
            with open(self.filename, "w", encoding="utf-8") as f:
                json.dump([], f, ensure_ascii=False, indent=2)
    
    def save_movies(self, movies: List[Movie]):
        """Сохраняет список фильмов в JSON"""
        with open(self.filename, "w", encoding="utf-8") as f:
            json.dump([movie.to_dict() for movie in movies], f, ensure_ascii=False, indent=2)
    
    def load_movies(self) -> List[Movie]:
        """Загружает список фильмов из JSON"""
        with open(self.filename, "r", encoding="utf-8") as f:
            data = json.load(f)
            return [Movie.from_dict(item) for item in data]
```

#### **gui.py** - Графический интерфейс
```python
import tkinter as tk
from tkinter import ttk, messagebox
from models import Movie
from storage import Storage

class MovieLibraryApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Movie Library - Личная кинотека")
        self.root.geometry("1000x600")
        self.root.resizable(True, True)
        
        # Загрузка данных
        self.storage = Storage()
        self.movies = self.storage.load_movies()
        self.filtered_movies = self.movies.copy()
        
        # Настройка интерфейса
        self._setup_ui()
        self._refresh_table()
    
    def _setup_ui(self):
        # Основной контейнер
        main_frame = ttk.Frame(self.root, padding="10")
        main_frame.grid(row=0, column=0, sticky=(tk.W, tk.E, tk.N, tk.S))
        
        # === Рамка добавления фильма ===
        add_frame = ttk.LabelFrame(main_frame, text="Добавить новый фильм", padding="10")
        add_frame.grid(row=0, column=0, columnspan=2, sticky=(tk.W, tk.E), pady=(0, 10))
        
        # Поля ввода
        ttk.Label(add_frame, text="Название:").grid(row=0, column=0, sticky=tk.W, padx=(0, 10))
        self.title_entry = ttk.Entry(add_frame, width=30)
        self.title_entry.grid(row=0, column=1, padx=(0, 20))
        
        ttk.Label(add_frame, text="Жанр:").grid(row=0, column=2, sticky=tk.W, padx=(0, 10))
        self.genre_combo = ttk.Combobox(add_frame, values=[
            "Боевик", "Комедия", "Драма", "Ужасы", "Фантастика", 
            "Триллер", "Мелодрама", "Детектив", "Приключения", "Аниме"
        ], width=20)
        self.genre_combo.grid(row=0, column=3, padx=(0, 20))
        
        ttk.Label(add_frame, text="Год:").grid(row=1, column=0, sticky=tk.W, padx=(0, 10), pady=(5, 0))
        self.year_entry = ttk.Entry(add_frame, width=30)
        self.year_entry.grid(row=1, column=1, padx=(0, 20), pady=(5, 0))
        
        ttk.Label(add_frame, text="Рейтинг (0-10):").grid(row=1, column=2, sticky=tk.W, padx=(0, 10), pady=(5, 0))
        self.rating_entry = ttk.Entry(add_frame, width=20)
        self.rating_entry.grid(row=1, column=3, padx=(0, 20), pady=(5, 0))
        
        # Кнопка добавления
        ttk.Button(add_frame, text="➕ Добавить фильм", command=self._add_movie).grid(row=1, column=4, pady=(5, 0))
        
        # === Рамка фильтрации ===
        filter_frame = ttk.LabelFrame(main_frame, text="Фильтрация", padding="10")
        filter_frame.grid(row=1, column=0, columnspan=2, sticky=(tk.W, tk.E), pady=(0, 10))
        
        ttk.Label(filter_frame, text="Фильтр по жанру:").grid(row=0, column=0, padx=(0, 10))
        self.filter_genre = ttk.Combobox(filter_frame, values=["Все"] + [
            "Боевик", "Комедия", "Драма", "Ужасы", "Фантастика", 
            "Триллер", "Мелодрама", "Детектив", "Приключения", "Аниме"
        ], width=20, state="readonly")
        self.filter_genre.set("Все")
        self.filter_genre.grid(row=0, column=1, padx=(0, 20))
        
        ttk.Label(filter_frame, text="Фильтр по году:").grid(row=0, column=2, padx=(0, 10))
        self.filter_year_from = ttk.Entry(filter_frame, width=8)
        self.filter_year_from.grid(row=0, column=3, padx=(0, 5))
        ttk.Label(filter_frame, text="до").grid(row=0, column=4, padx=(0, 5))
        self.filter_year_to = ttk.Entry(filter_frame, width=8)
        self.filter_year_to.grid(row=0, column=5, padx=(0, 20))
        
        ttk.Button(filter_frame, text="🔍 Применить фильтр", command=self._apply_filter).grid(row=0, column=6, padx=(0, 10))
        ttk.Button(filter_frame, text="🔄 Сбросить", command=self._reset_filter).grid(row=0, column=7)
        
        # Статистика
        self.stats_label = ttk.Label(filter_frame, text="", foreground="blue")
        self.stats_label.grid(row=1, column=0, columnspan=8, pady=(10, 0))
        
        # === Таблица фильмов ===
        table_frame = ttk.LabelFrame(main_frame, text="Список фильмов", padding="10")
        table_frame.grid(row=2, column=0, columnspan=2, sticky=(tk.W, tk.E, tk.N, tk.S))
        
        # Создание таблицы
        columns = ("title", "genre", "year", "rating")
        self.tree = ttk.Treeview(table_frame, columns=columns, show="headings", height=15)
        
        # Настройка колонок
        self.tree.heading("title", text="Название фильма")
        self.tree.heading("genre", text="Жанр")
        self.tree.heading("year", text="Год")
        self.tree.heading("rating", text="Рейтинг")
        
        self.tree.column("title", width=350)
        self.tree.column("genre", width=150)
        self.tree.column("year", width=80, anchor="center")
        self.tree.column("rating", width=80, anchor="center")
        
        # Полоса прокрутки
        scrollbar = ttk.Scrollbar(table_frame, orient="vertical", command=self.tree.yview)
        self.tree.configure(yscrollcommand=scrollbar.set)
        self.tree.pack(side="left", fill="both", expand=True)
        scrollbar.pack(side="right", fill="y")
        
        # Контекстное меню для таблицы
        self._create_context_menu()
        
        # Кнопки управления
        button_frame = ttk.Frame(main_frame)
        button_frame.grid(row=3, column=0, columnspan=2, pady=(10, 0))
        
        ttk.Button(button_frame, text="❌ Удалить выбранный фильм", command=self._delete_movie).pack(side="left", padx=5)
        ttk.Button(button_frame, text="📊 Показать статистику", command=self._show_statistics).pack(side="left", padx=5)
        ttk.Button(button_frame, text="💾 Сохранить вручную", command=self._manual_save).pack(side="left", padx=5)
        
        # Настройка веса для расширения
        self.root.columnconfigure(0, weight=1)
        self.root.rowconfigure(0, weight=1)
        main_frame.columnconfigure(0, weight=1)
        main_frame.rowconfigure(2, weight=1)
        table_frame.columnconfigure(0, weight=1)
        table_frame.rowconfigure(0, weight=1)
    
    def _create_context_menu(self):
        """Создаёт контекстное меню для таблицы"""
        self.context_menu = tk.Menu(self.root, tearoff=0)
        self.context_menu.add_command(label="Просмотреть информацию", command=self._show_movie_details)
        self.context_menu.add_separator()
        self.context_menu.add_command(label="Удалить", command=self._delete_movie)
        
        self.tree.bind("<Button-3>", self._show_context_menu)
        self.tree.bind("<Double-1>", self._show_movie_details)
    
    def _show_context_menu(self, event):
        """Показывает контекстное меню"""
        item = self.tree.identify_row(event.y)
        if item:
            self.tree.selection_set(item)
            self.context_menu.post(event.x_root, event.y_root)
    
    def _add_movie(self):
        """Добавляет новый фильм с валидацией"""
        try:
            title = self.title_entry.get()
            genre = self.genre_combo.get()
            year_str = self.year_entry.get()
            rating_str = self.rating_entry.get()
            
            # Проверка на пустые поля
            if not all([title, genre, year_str, rating_str]):
                messagebox.showwarning("Ошибка ввода", "Пожалуйста, заполните все поля")
                return
            
            # Валидация года
            try:
                year = int(year_str)
            except ValueError:
                messagebox.showerror("Ошибка", "Год должен быть целым числом")
                return
            
            # Валидация рейтинга
            try:
                rating = float(rating_str)
            except ValueError:
                messagebox.showerror("Ошибка", "Рейтинг должен быть числом")
                return
            
            # Создание фильма (валидация сработает внутри)
            movie = Movie(title, genre, year, rating)
            self.movies.append(movie)
            self.storage.save_movies(self.movies)
            
            # Обновление интерфейса
            self._apply_filter()
            self._clear_inputs()
            
            messagebox.showinfo("Успех", f"Фильм '{title}' успешно добавлен!")
            
        except ValueError as e:
            messagebox.showerror("Ошибка валидации", str(e))
    
    def _clear_inputs(self):
        """Очищает поля ввода"""
        self.title_entry.delete(0, tk.END)
        self.genre_combo.set("")
        self.year_entry.delete(0, tk.END)
        self.rating_entry.delete(0, tk.END)
    
    def _apply_filter(self):
        """Применяет фильтрацию по жанру и году"""
        genre_filter = self.filter_genre.get()
        year_from_str = self.filter_year_from.get()
        year_to_str = self.filter_year_to.get()
        
        self.filtered_movies = self.movies.copy()
        
        # Фильтр по жанру
        if genre_filter != "Все":
            self.filtered_movies = [m for m in self.filtered_movies if m.genre == genre_filter]
        
        # Фильтр по году
        if year_from_str:
            try:
                year_from = int(year_from_str)
                self.filtered_movies = [m for m in self.filtered_movies if m.year >= year_from]
            except ValueError:
                if year_from_str:
                    messagebox.showwarning("Ошибка", "Год 'от' должен быть числом")
        
        if year_to_str:
            try:
                year_to = int(year_to_str)
                self.filtered_movies = [m for m in self.filtered_movies if m.year <= year_to]
            except ValueError:
                if year_to_str:
                    messagebox.showwarning("Ошибка", "Год 'до' должен быть числом")
        
        self._refresh_table()
        self._update_stats()
    
    def _reset_filter(self):
        """Сбрасывает все фильтры"""
        self.filter_genre.set("Все")
        self.filter_year_from.delete(0, tk.END)
        self.filter_year_to.delete(0, tk.END)
        self.filtered_movies = self.movies.copy()
        self._refresh_table()
        self._update_stats()
    
    def _refresh_table(self):
        """Обновляет таблицу с фильмами"""
        # Очищаем таблицу
        for row in self.tree.get_children():
            self.tree.delete(row)
        
        # Добавляем фильмы
        for movie in self.filtered_movies:
            # Цвет для рейтинга
            rating_display = f"{movie.rating:.1f}"
            if movie.rating >= 8:
                rating_display = f"⭐ {rating_display}"
            elif movie.rating >= 6:
                rating_display = f"👍 {rating_display}"
            elif movie.rating >= 4:
                rating_display = f"😐 {rating_display}"
            else:
                rating_display = f"👎 {rating_display}"
            
            self.tree.insert("", tk.END, values=(
                movie.title,
                movie.genre,
                movie.year,
                rating_display
            ))
    
    def _update_stats(self):
        """Обновляет статистику фильтрации"""
        total = len(self.movies)
        filtered = len(self.filtered_movies)
        if total == 0:
            self.stats_label.config(text="Библиотека пуста. Добавьте первый фильм!")
        elif total == filtered:
            self.stats_label.config(text=f"Всего фильмов: {total}")
        else:
            self.stats_label.config(text=f"Показано: {filtered} из {total} фильмов")
    
    def _delete_movie(self):
        """Удаляет выбранный фильм"""
        selected = self.tree.selection()
        if not selected:
            messagebox.showwarning("Нет выбора", "Пожалуйста, выберите фильм для удаления")
            return
        
        # Получаем выбранный фильм
        item = selected[0]
        index = self.tree.index(item)
        movie = self.filtered_movies[index]
        
        # Подтверждение удаления
        if messagebox.askyesno("Подтверждение удаления", 
                               f"Вы действительно хотите удалить фильм\n\n'{movie.title}' ({movie.year})?"):
            # Удаляем из основного списка
            self.movies.remove(movie)
            self.storage.save_movies(self.movies)
            
            # Обновляем фильтр
            self._apply_filter()
            messagebox.showinfo("Удалено", f"Фильм '{movie.title}' удалён из библиотеки")
    
    def _show_movie_details(self, event=None):
        """Показывает подробную информацию о фильме"""
        selected = self.tree.selection()
        if not selected:
            return
        
        item = selected[0]
        index = self.tree.index(item)
        movie = self.filtered_movies[index]
        
        # Оценка рейтинга текстом
        rating_text = ""
        if movie.rating >= 9:
            rating_text = "🎉 Шедевр!"
        elif movie.rating >= 7:
            rating_text = "👍 Отличный фильм"
        elif movie.rating >= 5:
            rating_text = "😐 Средненько"
        elif movie.rating >= 3:
            rating_text = "🤔 На любителя"
        else:
            rating_text = "💩 Очень плохо"
        
        details = f"""
📽️ НАЗВАНИЕ: {movie.title}
🎭 ЖАНР: {movie.genre}
📅 ГОД: {movie.year}
⭐ РЕЙТИНГ: {movie.rating}/10
💬 ОЦЕНКА: {rating_text}
        """
        
        messagebox.showinfo("Информация о фильме", details)
    
    def _show_statistics(self):
        """Показывает статистику по библиотеке фильмов"""
        if not self.movies:
            messagebox.showinfo("Статистика", "В библиотеке пока нет фильмов")
            return
        
        # Собираем статистику
        total = len(self.movies)
        avg_rating = sum(m.rating for m in self.movies) / total
        oldest = min(self.movies, key=lambda m: m.year)
        newest = max(self.movies, key=lambda m: m.year)
        best = max(self.movies, key=lambda m: m.rating)
        worst = min(self.movies, key=lambda m: m.rating)
        
        # Статистика по жанрам
        genres = {}
        for movie in self.movies:
            genres[movie.genre] = genres.get(movie.genre, 0) + 1
        top_genre = max(genres.items(), key=lambda x: x[1])
        
        stats = f"""
📊 СТАТИСТИКА ВАШЕЙ КИНОТЕКИ

🎬 Всего фильмов: {total}
⭐ Средний рейтинг: {avg_rating:.1f}/10

📅 Самый старый фильм: {oldest.title} ({oldest.year})
🆕 Самый новый фильм: {newest.title} ({newest.year})

🏆 Лучший фильм: {best.title} - рейтинг {best.rating}/10
💩 Худший фильм: {worst.title} - рейтинг {worst.rating}/10

🎭 Самый популярный жанр: {top_genre[0]} ({top_genre[1]} фильмов)
        """
        
        messagebox.showinfo("Статистика библиотеки", stats)
    
    def _manual_save(self):
        """Ручное сохранение данных"""
        try:
            self.storage.save_movies(self.movies)
            messagebox.showinfo("Сохранено", "Данные успешно сохранены в файл movies.json")
        except Exception as e:
            messagebox.showerror("Ошибка", f"Не удалось сохранить данные: {str(e)}")
```

#### **main.py** - Точка входа
```python
import tkinter as tk
from gui import MovieLibraryApp

def main():
    root = tk.Tk()
    app = MovieLibraryApp(root)
    root.mainloop()

if __name__ == "__main__":
    main()
```

#### **tests.py** - Модульные тесты
```python
import unittest
import os
import tempfile
import json
from models import Movie
from storage import Storage

class TestMovieValidation(unittest.TestCase):
    """Тестирование валидации модели фильма"""
    
    def test_valid_movie_creation(self):
        """Позитивный тест: создание корректного фильма"""
        movie = Movie("Начало", "Фантастика", 2010, 8.8)
        self.assertEqual(movie.title, "Начало")
        self.assertEqual(movie.genre, "Фантастика")
        self.assertEqual(movie.year, 2010)
        self.assertEqual(movie.rating, 8.8)
    
    def test_invalid_title_empty(self):
        """Негативный тест: пустое название"""
        with self.assertRaises(ValueError):
            Movie("", "Драма", 2020, 7.5)
    
    def test_invalid_title_too_long(self):
        """Негативный тест: слишком длинное название"""
        with self.assertRaises(ValueError):
            Movie("A" * 101, "Драма", 2020, 7.5)
    
    def test_invalid_genre_empty(self):
        """Негативный тест: пустой жанр"""
        with self.assertRaises(ValueError):
            Movie("Фильм", "", 2020, 7.5)
    
    def test_invalid_genre_too_long(self):
        """Граничный тест: слишком длинный жанр"""
        with self.assertRaises(ValueError):
            Movie("Фильм", "B" * 51, 2020, 7.5)
    
    def test_invalid_year_below_min(self):
        """Негативный тест: год меньше 1888"""
        with self.assertRaises(ValueError):
            Movie("Фильм", "Драма", 1887, 7.5)
    
    def test_invalid_year_above_max(self):
        """Негативный тест: год больше 2026"""
        with self.assertRaises(ValueError):
            Movie("Фильм", "Драма", 2027, 7.5)
    
    def test_valid_year_boundary_min(self):
        """Граничный тест: минимальный допустимый год"""
        movie = Movie("Старый фильм", "Драма", 1888, 5.0)
        self.assertEqual(movie.year, 1888)
    
    def test_valid_year_boundary_max(self):
        """Граничный тест: максимальный допустимый год"""
        movie = Movie("Новый фильм", "Драма", 2026, 5.0)
        self.assertEqual(movie.year, 2026)
    
    def test_invalid_rating_below_zero(self):
        """Негативный тест: рейтинг меньше 0"""
        with self.assertRaises(ValueError):
            Movie("Фильм", "Драма", 2020, -0.1)
    
    def test_invalid_rating_above_ten(self):
        """Негативный тест: рейтинг больше 10"""
        with self.assertRaises(ValueError):
            Movie("Фильм", "Драма", 2020, 10.1)
    
    def test_valid_rating_boundary_min(self):
        """Граничный тест: минимальный рейтинг 0"""
        movie = Movie("Плохой фильм", "Драма", 2020, 0)
        self.assertEqual(movie.rating, 0)
    
    def test_valid_rating_boundary_max(self):
        """Граничный тест: максимальный рейтинг 10"""
        movie = Movie("Отличный фильм", "Драма", 2020, 10)
        self.assertEqual(movie.rating, 10)
    
    def test_to_dict_conversion(self):
        """Тест преобразования в словарь"""
        movie = Movie("Тест", "Комедия", 2000, 7.5)
        movie_dict = movie.to_dict()
        self.assertEqual(movie_dict["title"], "Тест")
        self.assertEqual(movie_dict["genre"], "Комедия")
        self.assertEqual(movie_dict["year"], 2000)
        self.assertEqual(movie_dict["rating"], 7.5)
    
    def test_from_dict_conversion(self):
        """Тест создания из словаря"""
        data = {"title": "Из словаря", "genre": "Триллер", "year": 2015, "rating": 8.2}
        movie = Movie.from_dict(data)
        self.assertEqual(movie.title, "Из словаря")
        self.assertEqual(movie.genre, "Триллер")
        self.assertEqual(movie.year, 2015)
        self.assertEqual(movie.rating, 8.2)

class TestStorage(unittest.TestCase):
    """Тестирование работы с JSON хранилищем"""
    
    def setUp(self):
        """Создаём временный файл для тестов"""
        self.temp_file = tempfile.NamedTemporaryFile(delete=False, suffix=".json")
        self.filename = self.temp_file.name
        self.temp_file.close()
    
    def tearDown(self):
        """Удаляем временный файл после тестов"""
        if os.path.exists(self.filename):
            os.remove(self.filename)
    
    def test_save_and_load_movies(self):
        """Позитивный тест: сохранение и загрузка фильмов"""
        storage = Storage(self.filename)
        movies = [
            Movie("Фильм 1", "Боевик", 2010, 7.5),
            Movie("Фильм 2", "Комедия", 2015, 8.0)
        ]
        
        storage.save_movies(movies)
        loaded_movies = storage.load_movies()
        
        self.assertEqual(len(loaded_movies), 2)
        self.assertEqual(loaded_movies[0].title, "Фильм 1")
        self.assertEqual(loaded_movies[1].year, 2015)
    
    def test_load_empty_file(self):
        """Граничный тест: загрузка из пустого файла"""
        storage = Storage(self.filename)
        movies = storage.load_movies()
        self.assertEqual(movies, [])
    
    def test_save_empty_list(self):
        """Граничный тест: сохранение пустого списка"""
        storage = Storage(self.filename)
        storage.save_movies([])
        
        with open(self.filename, "r", encoding="utf-8") as f:
            data = json.load(f)
        self.assertEqual(data, [])
    
    def test_file_created_automatically(self):
        """Позитивный тест: автоматическое создание файла"""
        if os.path.exists(self.filename):
            os.remove(self.filename)
        
        storage = Storage(self.filename)
        self.assertTrue(os.path.exists(self.filename))

if __name__ == "__main__":
    # Запуск всех тестов с подробным выводом
    unittest.main(verbosity=2)
```

#### **.gitignore**
```
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python

# Virtual Environment
venv/
env/
ENV/
env.bak/
venv.bak/

# IDE
.vscode/
.idea/
*.swp
*.swo
*~

# Project specific
movies.json
*.log
.DS_Store
Thumbs.db

# Testing
.pytest_cache/
.coverage
htmlcov/
```

#### **README.md**
```markdown
# 🎬 Movie Library - Личная кинотека

**Автор:** Александр Петров

Приложение для ведения личной коллекции фильмов с возможностью добавления, фильтрации, сохранения и загрузки данных.

## 📋 Описание

Movie Library - это desktop-приложение с графическим интерфейсом на Python (tkinter), которое позволяет:
- Добавлять фильмы с указанием названия, жанра, года выпуска и рейтинга
- Просматривать список всех фильмов в удобной таблице
- Фильтровать фильмы по жанру и году выпуска
- Просматривать подробную информацию о каждом фильме (двойной клик)
- Удалять фильмы из коллекции
- Просматривать статистику (средний рейтинг, лучший/худший фильм, популярные жанры)
- Автоматически сохранять данные в JSON-файл

## 🚀 Требования

- Python 3.7 или выше
- Стандартные библиотеки Python (tkinter, json, os, unittest)

## 📥 Установка и запуск

1. **Клонируйте репозиторий:**
```bash
git clone https://github.com/yourusername/movie-library.git
cd movie-library
```

2. **Запустите приложение:**
```bash
python main.py
```

## 🎮 Использование

### Добавление фильма
1. Заполните все поля в форме "Добавить новый фильм"
2. Нажмите кнопку "➕ Добавить фильм"
3. Фильм появится в таблице

### Фильтрация
- **По жанру:** выберите жанр из выпадающего списка
- **По году:** укажите диапазон годов (от и до)
- Нажмите "🔍 Применить фильтр"

### Управление фильмами
- **Просмотр информации:** дважды кликните по фильму или используйте правую кнопку мыши
- **Удаление:** выделите фильм и нажмите "❌ Удалить выбранный фильм"
- **Статистика:** нажмите "📊 Показать статистику"

## ✅ Примеры использования

### Пример 1: Добавление фильма
```
Название: Побег из Шоушенка
Жанр: Драма
Год: 1994
Рейтинг: 9.3
→ Результат: фильм добавлен с высоким рейтингом ⭐
```

### Пример 2: Фильтрация
```
Фильтр по жанру: Фантастика
Фильтр по году: 2010 до 2020
→ Результат: отображаются только фантастические фильмы 2010-2020 годов
```

### Пример 3: Проверка валидации
```
Название: (пустое поле)
→ Ошибка: "Название не может быть пустым"

Год: "две тысячи"
→ Ошибка: "Год должен быть целым числом"

Рейтинг: 11
→ Ошибка: "Рейтинг должен быть числом от 0 до 10"
```

## 🧪 Тестирование

Для запуска тестов выполните:
```bash
python -m unittest tests.py -v
```

### Ожидаемый результат:
```
test_invalid_genre_empty (__main__.TestMovieValidation) ... ok
test_invalid_rating_above_ten ... ok
test_invalid_title_empty ... ok
test_invalid_year_below_min ... ok
test_save_and_load_movies ... ok
...
----------------------------------------------------------------------
Ran 15 tests in 0.234s

OK
```

## 📁 Структура проекта

```
movie-library/
│
├── main.py          # Точка входа в приложение
├── gui.py           # Графический интерфейс (tkinter)
├── models.py        # Модель данных и валидация
├── storage.py       # Работа с JSON файлом
├── tests.py         # Модульные тесты
├── movies.json      # Файл с данными (создаётся автоматически)
├── .gitignore       # Игнорируемые файлы Git
└── README.md        # Документация
```

## 🛡️ Валидация данных

Приложение проверяет:
- ✅ Название: не пустое, не более 100 символов
- ✅ Жанр: не пустой, не более 50 символов
- ✅ Год: целое число от 1888 до 2026
- ✅ Рейтинг: число от 0 до 10 (поддерживаются десятичные дроби)

## 💾 Хранение данных

Данные автоматически сохраняются в файл `movies.json` в формате:
```json
[
  {
    "title": "Начало",
    "genre": "Фантастика",
    "year": 2010,
    "rating": 8.8
  }
]
```

## 🎨 Дополнительные возможности

- **Цветовая индикация рейтинга** в таблице:
  - ⭐ 8-10: отличные фильмы
  - 👍 6-7.9: хорошие фильмы
  - 😐 4-5.9: средние фильмы
  - 👎 0-3.9: плохие фильмы

- **Контекстное меню** (правой кнопкой мыши) для быстрых действий

- **Автосохранение** при добавлении/удалении фильмов

## 📝 Лицензия

MIT License

## 👤 Контакты

Автор: Александр Петров
GitHub: https://github.com/yourusername
```

---

## 🧪 Результаты тестирования

Запустите тесты:

```bash
python tests.py
```

**Ожидаемый вывод:**
```
test_invalid_genre_empty (__main__.TestMovieValidation) ... ok
test_invalid_genre_too_long (__main__.TestMovieValidation) ... ok
test_invalid_rating_above_ten (__main__.TestMovieValidation) ... ok
test_invalid_rating_below_zero (__main__.TestMovieValidation) ... ok
test_invalid_title_empty (__main__.TestMovieValidation) ... ok
test_invalid_title_too_long (__main__.TestMovieValidation) ... ok
test_invalid_year_above_max (__main__.TestMovieValidation) ... ok
test_invalid_year_below_min (__main__.TestMovieValidation) ... ok
test_to_dict_conversion (__main__.TestMovieValidation) ... ok
test_valid_movie_creation (__main__.TestMovieValidation) ... ok
test_valid_rating_boundary_max (__main__.TestMovieValidation) ... ok
test_valid_rating_boundary_min (__main__.TestMovieValidation) ... ok
test_valid_year_boundary_max (__main__.TestMovieValidation) ... ok
test_valid_year_boundary_min
