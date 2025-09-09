<Window x:Class="WpfTestingClient.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Тестирование" Height="450" Width="800">

    <Grid Margin="10">
        <StackPanel>
            <!-- Вопрос -->
            <TextBlock x:Name="TxtQuestion" 
                       Text="Вопрос появится здесь"
                       FontSize="18"
                       TextWrapping="Wrap"
                       Margin="0,0,0,15"/>
            
            <!-- Варианты -->
            <ListBox x:Name="LstOptions" 
                     SelectionMode="Extended"
                     Height="200"
                     Margin="0,0,0,15"
                     DisplayMemberPath="Text"/>
            
            <!-- Для текстового ответа -->
            <TextBox x:Name="TxtAnswer"
                     Height="30"
                     Visibility="Visible"
                     Margin="0,0,0,15"/>
            
            <!-- Кнопка -->
            <Button x:Name="BtnNext"
                    Content="Дальше"
                    Height="40"
                    Click="BtnNext_Click"/>
        </StackPanel>
    </Grid>
</Window>
