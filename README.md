<Window x:Class="WpfClient.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Клиент товаров" Height="600" Width="900"
        WindowStartupLocation="CenterScreen">
    <Grid Margin="10">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>
        
        <!-- Панель управления -->
        <StackPanel Grid.Row="0" Orientation="Horizontal" Margin="0,0,0,10">
            <Button Content="📦 Список товаров" Click="GetList_Click" 
                    Padding="15,5" FontSize="14" Background="#FF4CAF50" Foreground="White"/>
            <Button Content="🔄 Обновить" Click="Refresh_Click" 
                    Padding="15,5" FontSize="14" Margin="10,0,0,0" Background="#FF2196F3" Foreground="White"/>
            <TextBlock Text="Статус:" VerticalAlignment="Center" Margin="20,0,5,0" FontWeight="Bold"/>
            <TextBlock x:Name="StatusText" Text="Не подключено" VerticalAlignment="Center" Foreground="Red"/>
        </StackPanel>
        
        <!-- Панель команд -->
        <GroupBox Grid.Row="1" Header="Команды серверу" Margin="0,0,0,10">
            <StackPanel Orientation="Horizontal">
                <TextBox x:Name="InputBox" TextWrapping="Wrap" MinWidth="300" 
                        Padding="5" FontSize="14" VerticalAlignment="Center"
                        ToolTip="Доступные команды: list, get 1, add, update, delete, categories"/>
                <Button Content="📤 Отправить" Click="SendCommand_Click" 
                        Padding="15,5" FontSize="14" Margin="10,0,0,0" Background="#FF9C27B0" Foreground="White"/>
            </StackPanel>
        </GroupBox>
        
        <!-- Отображение данных -->
        <TabControl Grid.Row="2" Margin="0,0,0,10">
            <TabItem Header="📋 Таблица товаров">
                <DataGrid x:Name="ProductsGrid" AutoGenerateColumns="False" 
                         IsReadOnly="True" SelectionMode="Single"
                         CanUserAddRows="False" CanUserDeleteRows="False">
                    <DataGrid.Columns>
                        <DataGridTextColumn Header="ID" Binding="{Binding Id}" Width="50"/>
                        <DataGridTextColumn Header="Название" Binding="{Binding Name}" Width="200"/>
                        <DataGridTextColumn Header="Описание" Binding="{Binding Description}" Width="250"/>
                        <DataGridTextColumn Header="Цена" Binding="{Binding Price, StringFormat=C}" Width="80"/>
                        <DataGridTextColumn Header="Категория" Binding="{Binding Category.Name}" Width="120"/>
                        <DataGridTextColumn Header="Количество" Binding="{Binding StockQuantity}" Width="80"/>
                        <DataGridTextColumn Header="Дата" Binding="{Binding CreatedDate, StringFormat=dd.MM.yyyy}" Width="100"/>
                    </DataGrid.Columns>
                </DataGrid>
            </TabItem>
            <TabItem Header="📄 Текстовый вывод">
                <TextBox x:Name="OutputText" TextWrapping="Wrap" IsReadOnly="True" 
                        VerticalScrollBarVisibility="Auto" HorizontalScrollBarVisibility="Auto"
                        FontFamily="Consolas" FontSize="12" Background="#FFF5F5F5"/>
            </TabItem>
        </TabControl>
        
        <!-- Информация о выбранном товаре -->
        <GroupBox Grid.Row="3" Header="🔍 Детали выбранного товара">
            <ScrollViewer MaxHeight="150" VerticalScrollBarVisibility="Auto">
                <StackPanel>
                    <TextBlock x:Name="SelectedProductText" TextWrapping="Wrap" Margin="5"/>
                </StackPanel>
            </ScrollViewer>
        </GroupBox>
    </Grid>
</Window>
