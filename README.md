using System;
using System.Collections.Generic;
using System.IO;
using System.Net.Sockets;
using System.Text;
using System.Text.Json;
using System.Windows;
using System.Windows.Controls;

namespace WpfClient
{
    public partial class MainWindow : Window
    {
        private const string ServerIP = "127.0.0.1";
        private const int ServerPort = 5000;

        public MainWindow()
        {
            InitializeComponent();
            StatusText.Text = "Готов к подключению";
            StatusText.Foreground = System.Windows.Media.Brushes.Orange;
            
            // Подсказка для пользователя
            InputBox.Text = "list";
            
            // Обработчик выбора товара в таблице
            ProductsGrid.SelectionChanged += ProductsGrid_SelectionChanged;
        }

        private void GetList_Click(object sender, RoutedEventArgs e)
        {
            try
            {
                string response = SendToServer("list");
                ProcessListResponse(response);
                StatusText.Text = "Данные загружены";
                StatusText.Foreground = System.Windows.Media.Brushes.Green;
            }
            catch (Exception ex)
            {
                ShowError($"Ошибка при получении списка: {ex.Message}");
            }
        }

        private void Refresh_Click(object sender, RoutedEventArgs e)
        {
            GetList_Click(sender, e);
        }

        private void SendCommand_Click(object sender, RoutedEventArgs e)
        {
            string command = InputBox.Text.Trim();
            if (string.IsNullOrEmpty(command))
            {
                MessageBox.Show("Введите команду", "Предупреждение", MessageBoxButton.OK, MessageBoxImage.Warning);
                return;
            }

            try
            {
                string response = SendToServer(command);
                
                // Автоматически определяем тип команды и обрабатываем ответ
                if (command.ToLower() == "list" || command.ToLower().StartsWith("get"))
                {
                    ProcessListResponse(response);
                }
                else
                {
                    OutputText.Text = FormatJsonResponse(response);
                }
                
                StatusText.Text = "Команда выполнена";
                StatusText.Foreground = System.Windows.Media.Brushes.Green;
            }
            catch (Exception ex)
            {
                ShowError($"Ошибка при выполнении команды: {ex.Message}");
            }
        }

        private void ProductsGrid_SelectionChanged(object sender, SelectionChangedEventArgs e)
        {
            if (ProductsGrid.SelectedItem is Product selectedProduct)
            {
                SelectedProductText.Text = $"ID: {selectedProduct.Id}\n" +
                                          $"Название: {selectedProduct.Name}\n" +
                                          $"Описание: {selectedProduct.Description}\n" +
                                          $"Цена: {selectedProduct.Price:C}\n" +
                                          $"Категория: {selectedProduct.Category?.Name ?? "N/A"}\n" +
                                          $"Количество: {selectedProduct.StockQuantity}\n" +
                                          $"Дата создания: {selectedProduct.CreatedDate:dd.MM.yyyy HH:mm}";
            }
        }

        private string SendToServer(string message)
        {
            try
            {
                using (TcpClient client = new TcpClient())
                {
                    // Таймаут подключения 5 секунд
                    if (!client.ConnectAsync(ServerIP, ServerPort).Wait(5000))
                    {
                        throw new TimeoutException("Превышено время ожидания подключения к серверу");
                    }

                    using (NetworkStream stream = client.GetStream())
                    {
                        byte[] data = Encoding.UTF8.GetBytes(message);
                        stream.Write(data, 0, data.Length);

                        // Читаем ответ с таймаутом
                        stream.ReadTimeout = 10000;
                        return ReadFullResponse(stream);
                    }
                }
            }
            catch (SocketException ex)
            {
                throw new Exception($"Не удалось подключиться к серверу: {ex.Message}");
            }
        }

        private string ReadFullResponse(NetworkStream stream)
        {
            byte[] buffer = new byte[4096];
            StringBuilder responseBuilder = new StringBuilder();
            int bytesRead;

            // Читаем все данные до конца
            while ((bytesRead = stream.Read(buffer, 0, buffer.Length)) > 0)
            {
                responseBuilder.Append(Encoding.UTF8.GetString(buffer, 0, bytesRead));
                
                // Проверяем, есть ли еще данные (неблокирующая проверка)
                if (stream.DataAvailable == false)
                    break;
            }

            return responseBuilder.ToString();
        }

        private void ProcessListResponse(string response)
        {
            try
            {
                // Пытаемся парсить как JSON
                var products = JsonSerializer.Deserialize<List<Product>>(response, new JsonSerializerOptions
                {
                    PropertyNameCaseInsensitive = true
                });

                ProductsGrid.ItemsSource = products;
                OutputText.Text = $"✅ Получено {products?.Count ?? 0} товаров\n\n" + FormatJsonResponse(response);
            }
            catch (JsonException)
            {
                // Если не JSON, показываем как plain text
                ProductsGrid.ItemsSource = null;
                OutputText.Text = response;
            }
        }

        private string FormatJsonResponse(string json)
        {
            try
            {
                using (JsonDocument document = JsonDocument.Parse(json))
                {
                    return JsonSerializer.Serialize(document.RootElement, new JsonSerializerOptions
                    {
                        WriteIndented = true
                    });
                }
            }
            catch
            {
                return json; // Возвращаем как есть, если не JSON
            }
        }

        private void ShowError(string message)
        {
            StatusText.Text = "Ошибка";
            StatusText.Foreground = System.Windows.Media.Brushes.Red;
            OutputText.Text = $"❌ {message}";
            MessageBox.Show(message, "Ошибка", MessageBoxButton.OK, MessageBoxImage.Error);
        }
    }

    // Модели данных для десериализации JSON
    public class Product
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public string Description { get; set; }
        public decimal Price { get; set; }
        public int CategoryId { get; set; }
        public Category Category { get; set; }
        public int StockQuantity { get; set; }
        public DateTime CreatedDate { get; set; }
    }

    public class Category
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public string Description { get; set; }
    }
}
