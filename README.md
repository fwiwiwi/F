-- 1. Таблица ролей
CREATE TABLE roles (
    role_id INT IDENTITY(1,1) PRIMARY KEY,
    role_name NVARCHAR(50) NOT NULL UNIQUE
);

-- 2. Таблица пользователей
CREATE TABLE users (
    user_id INT IDENTITY(1,1) PRIMARY KEY,
    username NVARCHAR(100) NOT NULL UNIQUE,
    email NVARCHAR(255) NOT NULL UNIQUE,
    password_hash NVARCHAR(255) NOT NULL,
    role_id INT NOT NULL FOREIGN KEY REFERENCES roles(role_id) ON DELETE CASCADE,
    created_at DATETIME2 DEFAULT GETDATE()
);

-- 3. Таблица курсов
CREATE TABLE courses (
    course_id INT IDENTITY(1,1) PRIMARY KEY,
    title NVARCHAR(255) NOT NULL,
    description NVARCHAR(MAX),
    created_at DATETIME2 DEFAULT GETDATE(),
    updated_at DATETIME2 DEFAULT GETDATE()
);

-- 4. Таблица модулей
CREATE TABLE modules (
    module_id INT IDENTITY(1,1) PRIMARY KEY,
    course_id INT NOT NULL FOREIGN KEY REFERENCES courses(course_id) ON DELETE CASCADE,
    title NVARCHAR(255) NOT NULL,
    description NVARCHAR(MAX),
    order_index INT NOT NULL,
    created_at DATETIME2 DEFAULT GETDATE()
);

-- 5. Таблица уроков
CREATE TABLE lessons (
    lesson_id INT IDENTITY(1,1) PRIMARY KEY,
    module_id INT NOT NULL FOREIGN KEY REFERENCES modules(module_id) ON DELETE CASCADE,
    title NVARCHAR(255) NOT NULL,
    description NVARCHAR(MAX),
    order_index INT NOT NULL,
    created_at DATETIME2 DEFAULT GETDATE()
);

-- 6. Таблица контента
CREATE TABLE content (
    content_id INT IDENTITY(1,1) PRIMARY KEY,
    lesson_id INT NOT NULL FOREIGN KEY REFERENCES lessons(lesson_id) ON DELETE CASCADE,
    title NVARCHAR(255) NOT NULL,
    content NVARCHAR(MAX),
    file_path NVARCHAR(500),
    content_type NVARCHAR(50) NOT NULL,
    order_index INT NOT NULL,
    created_at DATETIME2 DEFAULT GETDATE()
);

-- 7. Таблица вопросов
CREATE TABLE questions (
    question_id INT IDENTITY(1,1) PRIMARY KEY,
    lesson_id INT NOT NULL FOREIGN KEY REFERENCES lessons(lesson_id) ON DELETE CASCADE,
    question_text NVARCHAR(MAX) NOT NULL,
    question_type NVARCHAR(50) DEFAULT 'single_choice',
    created_at DATETIME2 DEFAULT GETDATE()
);

-- Таблица ответов
CREATE TABLE answers (
    answer_id INT IDENTITY(1,1) PRIMARY KEY,
    question_id INT NOT NULL FOREIGN KEY REFERENCES questions(question_id) ON DELETE CASCADE,
    answer_text NVARCHAR(MAX) NOT NULL,
    is_correct BIT NOT NULL DEFAULT 0
);

-- Таблица прогресса
CREATE TABLE user_progress (
    progress_id INT IDENTITY(1,1) PRIMARY KEY,
    user_id INT NOT NULL FOREIGN KEY REFERENCES users(user_id) ON DELETE CASCADE,
    lesson_id INT NOT NULL FOREIGN KEY REFERENCES lessons(lesson_id) ON DELETE CASCADE,
    is_completed BIT DEFAULT 0,
    time_spent INT DEFAULT 0,
    last_visited DATETIME2,
    CONSTRAINT UK_user_lesson UNIQUE (user_id, lesson_id)
);

-- Таблица результатов тестов
CREATE TABLE quiz_results (
    result_id INT IDENTITY(1,1) PRIMARY KEY,
    user_id INT NOT NULL FOREIGN KEY REFERENCES users(user_id) ON DELETE CASCADE,
    lesson_id INT NOT NULL FOREIGN KEY REFERENCES lessons(lesson_id) ON DELETE CASCADE,
    score INT NOT NULL,
    total_questions INT NOT NULL,
    started_at DATETIME2 DEFAULT GETDATE(),
    completed_at DATETIME2
);

-- Заполнение базовых данных
INSERT INTO roles (role_name) VALUES 
('student'),
('teacher'),
('admin');

-- Пароль: password123 (хэш bcrypt)
INSERT INTO users (username, email, password_hash, role_id) VALUES
('student1', 'student1@example.com', '$2b$10$N9qo8uLOickgx2ZMRZoMye3.ZG2LVDyS3g6Iqk7Q2F/JK4GYdOzO6', 1),
('teacher1', 'teacher1@example.com', '$2b$10$N9qo8uLOickgx2ZMRZoMye3.ZG2LVDyS3g6Iqk7Q2F/JK4GYdOzO6', 2);

INSERT INTO courses (title, description) VALUES
('Разработка на Kotlin', 'Изучение основ языка Kotlin для начинающих'),
('Базы данных PostgreSQL', 'Основы работы с СУБД PostgreSQL и SQL');

INSERT INTO modules (course_id, title, description, order_index) VALUES
(1, 'Введение в Kotlin', 'Базовые понятия и синтаксис языка', 1),
(1, 'ООП в Kotlin', 'Объектно-ориентированное программирование', 2),
(2, 'Основы SQL', 'Язык запросов SQL и работа с данными', 1);

INSERT INTO lessons (module_id, title, description, order_index) VALUES
(1, 'Первая программа', 'Создание Hello World на Kotlin', 1),
(1, 'Переменные и типы данных', 'Основные типы данных и объявление переменных', 2),
(2, 'Классы и объекты', 'Создание классов и объектов в Kotlin', 1),
(3, 'SELECT запросы', 'Основы выборки данных из таблиц', 1);

INSERT INTO content (lesson_id, title, content, content_type, order_index) VALUES
(1, 'Установка Kotlin', 'Для начала работы需要 установить JDK и Kotlin compiler...', 'text', 1),
(1, 'Hello World код', 'fun main() { println("Hello, World!") }', 'text', 2),
(2, 'Типы данных', 'Kotlin поддерживает Int, String, Boolean, Double и другие типы...', 'text', 1);

INSERT INTO questions (lesson_id, question_text, question_type) VALUES
(1, 'Как объявить функцию main в Kotlin?', 'single_choice'),
(1, 'Какая функция выводит текст в консоль?', 'single_choice'),
(2, 'Как объявить переменную в Kotlin?', 'single_choice');

INSERT INTO answers (question_id, answer_text, is_correct) VALUES
(1, 'fun main()', 1),
(1, 'function main()', 0),
(1, 'def main():', 0),
(1, 'void main()', 0),
(2, 'println()', 1),
(2, 'print()', 0),
(2, 'console.log()', 0),
(2, 'System.out.println()', 0),
(3, 'var x = 10', 1),
(3, 'variable x = 10', 0),
(3, 'int x = 10', 0),
(3, 'let x = 10', 0);

-- Создание индексов для улучшения производительности
CREATE INDEX IX_users_role_id ON users(role_id);
CREATE INDEX IX_modules_course_id ON modules(course_id);
CREATE INDEX IX_lessons_module_id ON lessons(module_id);
CREATE INDEX IX_content_lesson_id ON content(lesson_id);
CREATE INDEX IX_questions_lesson_id ON questions(lesson_id);
CREATE INDEX IX_answers_question_id ON answers(question_id);
CREATE INDEX IX_user_progress_user_lesson ON user_progress(user_id, lesson_id);
CREATE INDEX IX_quiz_results_user_lesson ON quiz_results(user_id, lesson_id);
