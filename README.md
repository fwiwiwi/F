using System;
using System.Collections.Generic;
using System.Linq;
using System.Windows;
using System.Windows.Controls;
using WpfTestingClient.Models;

namespace WpfTestingClient
{
    public partial class MainWindow : Window
    {
        private List<Question> _allQuestions = new();
        private List<Question> _currentQuestions = new();
        private int _currentIndex = 0;
        private int _score = 0;
        private readonly Random _rng = new();

        public MainWindow()
        {
            InitializeComponent();
            LoadQuestionsFromCode();
            StartTest();
        }

        private void LoadQuestionsFromCode()
        {
            _allQuestions = new List<Question>
            {
                new Question
                {
                    Text = "Что такое HTML?",
                    Kind = InteractiveKind.SingleChoice,
                    Hint = "Это язык разметки.",
                    Explanation = "HTML — язык гипертекстовой разметки.",
                    Options = new List<AnswerOption>
                    {
                        new AnswerOption { Text = "Язык разметки", IsCorrect = true },
                        new AnswerOption { Text = "Язык программирования", IsCorrect = false },
                        new AnswerOption { Text = "База данных", IsCorrect = false },
                        new AnswerOption { Text = "Операционная система", IsCorrect = false }
                    }
                },
                new Question
                {
                    Text = "Выберите правильные утверждения про C# (несколько).",
                    Kind = InteractiveKind.MultipleChoice,
                    Hint = "C# — язык от Microsoft.",
                    Explanation = "C# — современный объектно-ориентированный язык.",
                    Options = new List<AnswerOption>
                    {
                        new AnswerOption { Text = "Поддерживает ООП", IsCorrect = true },
                        new AnswerOption { Text = "Нельзя использовать LINQ", IsCorrect = false },
                        new AnswerOption { Text = "Есть garbage collector", IsCorrect = true },
                        new AnswerOption { Text = "Это функциональная БД", IsCorrect = false }
                    }
                },
                new Question
                {
                    Text = "Упорядочите шаги компиляции: 1) Выполнение 2) Компиляция 3) Написание кода",
                    Kind = InteractiveKind.Ordering,
                    Hint = "Сначала пишем код.",
                    Explanation = "Правильный порядок: Написание -> Компиляция -> Выполнение.",
                    Options = new List<AnswerOption>
                    {
                        new AnswerOption { Text = "Написание кода", IsCorrect = true },
                        new AnswerOption { Text = "Компиляция", IsCorrect = true },
                        new AnswerOption { Text = "Выполнение", IsCorrect = true },
                        new AnswerOption { Text = "(пусто)", IsCorrect = false }
                    }
                },
                new Question
                {
                    Text = "Введите номер телефона в формате +7-xxx-xxx-xx-xx",
                    Kind = InteractiveKind.TextInput,
                    Hint = "Пример: +7-123-456-78-90",
                    Explanation = "Формат: +7-XXX-XXX-XX-XX",
                    RegexAnswer = @"^\+7-\d{3}-\d{3}-\d{2}-\d{2}$"
                },
                new Question
                {
                    Text = "Сопоставьте столицу и страну.",
                    Kind = InteractiveKind.Matching,
                    Hint = "Москва — Россия.",
                    Explanation = "Москва - Россия, Берлин - Германия, Париж - Франция.",
                    Options = new List<AnswerOption>
                    {
                        new AnswerOption { Text = "Москва - Россия", IsCorrect = true },
                        new AnswerOption { Text = "Берлин - Германия", IsCorrect = true },
                        new AnswerOption { Text = "Париж - Франция", IsCorrect = true },
                        new AnswerOption { Text = "(пусто)", IsCorrect = false }
                    }
                }
            };
        }

        private void StartTest()
        {
            _score = 0;
            _currentIndex = 0;

            // выбираем случайные 10 вопросов
            _currentQuestions = _allQuestions
                .OrderBy(q => _rng.Next())
                .Take(10)
                .ToList();

            ShowQuestion();
        }

        private void ShowQuestion()
        {
            if (_currentIndex >= _currentQuestions.Count)
            {
                ShowResult();
                return;
            }

            var q = _currentQuestions[_currentIndex];
            TxtQuestion.Text = q.Text;
            LstOptions.ItemsSource = null;

            if (q.Options != null && q.Options.Any())
            {
                var shuffled = q.Options.OrderBy(o => _rng.Next()).ToList();
                LstOptions.ItemsSource = shuffled;
            }
        }

        private void BtnNext_Click(object sender, RoutedEventArgs e)
        {
            CheckAnswer();
            _currentIndex++;
            ShowQuestion();
        }

        private void CheckAnswer()
        {
            if (_currentIndex >= _currentQuestions.Count)
                return;

            var q = _currentQuestions[_currentIndex];

            if (q.Kind == InteractiveKind.SingleChoice)
            {
                if (LstOptions.SelectedItem is AnswerOption selected && selected.IsCorrect)
                    _score++;
            }
            else if (q.Kind == InteractiveKind.MultipleChoice)
            {
                var selected = LstOptions.SelectedItems.Cast<AnswerOption>().ToList();
                if (selected.All(o => o.IsCorrect) && selected.Count == q.Options.Count(o => o.IsCorrect))
                    _score++;
            }
            else if (q.Kind == InteractiveKind.TextInput)
            {
                if (!string.IsNullOrEmpty(TxtAnswer.Text) &&
                    System.Text.RegularExpressions.Regex.IsMatch(TxtAnswer.Text, q.RegexAnswer))
                {
                    _score++;
                }
            }
            // упрощённая проверка для Ordering и Matching
            else if (q.Kind == InteractiveKind.Ordering || q.Kind == InteractiveKind.Matching)
            {
                // считаем правильным, если выбрано хотя бы 3 правильных
                var selected = LstOptions.SelectedItems.Cast<AnswerOption>().ToList();
                if (selected.Count(o => o.IsCorrect) >= 3)
                    _score++;
            }
        }

        private void ShowResult()
        {
            int total = _currentQuestions.Count;
            double percent = (double)_score / total * 100;
            string grade;

            if (percent >= 90) grade = "Отлично";
            else if (percent >= 75) grade = "Хорошо";
            else if (percent >= 60) grade = "Удовлетворительно";
            else grade = "Неудовлетворительно";

            MessageBox.Show(
                $"Результаты:\n" +
                $"Правильных: {_score} из {total}\n" +
                $"Процент: {percent:F2}%\n" +
                $"Оценка: {grade}",
                "Тест завершён");
        }
    }
}
