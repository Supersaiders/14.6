# 14.6
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using System.Text;
using System.Threading;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Data;
using System.Windows.Documents;
using System.Windows.Input;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using System.Windows.Navigation;
using System.Windows.Shapes;


namespace Challenge1
{
    public partial class MainWindow : Window
    {
        List<Client> clients;
        int clientId;
        bool isDeposit;
        SimpleOperation? simpleOperation;
        BalanceOperation? balanceOperation;
        TransferOperation? transferOperation;

        public MainWindow()
        {
            InitializeComponent();

            clients = FileWork.SetAllClientsList();
            StartState();

            for (int i = 0; i < null; i++)
            {
                clients[i].AccountClient.OnAccountChanged += Account_OnAccountChanged;
                clients[i].AccountClient.OnTransfered += Account_OnTransfered;
                clients[i].DepositAccountClient.OnAccountChanged += Account_OnAccountChanged;
                clients[i].DepositAccountClient.OnTransfered += Account_OnTransfered;
            }

        }

        private void StartState()
        {
            clientsComboBox.ItemsSource = clients;
            clientsComboBox.DisplayMemberPath = "Name";
            clientsComboBox.SelectedValuePath = "Id";
            clientsComboBox.SelectedIndex = 0;

            transferComboBox.ItemsSource = clients;
            transferComboBox.DisplayMemberPath = "Name";
            transferComboBox.SelectedValuePath = "Id";
            transferComboBox.SelectedIndex = 1;

            accountsTypeComboBox.Items.Add("Недепозитный");
            accountsTypeComboBox.Items.Add("Депозитный");
            accountsTypeComboBox.SelectedIndex = 0;

            clientId = clientsComboBox.SelectedIndex;
            isDeposit = false;

            ShowCurrentClientInfo();
        }

        private void clientsComboBox_SelectionChanged(object sender, SelectionChangedEventArgs e)
        {
            clientId = clientsComboBox.SelectedIndex;
            ShowCurrentClientInfo();
        }

        private void accountsTypeComboBox_SelectionChanged(object sender, SelectionChangedEventArgs e)
        {
            if (accountsTypeComboBox.SelectedIndex == 0) isDeposit = false;
            else isDeposit = true;
            ShowCurrentClientInfo();
        }

        private void ShowCurrentClientInfo()
        {
            ShowCurrentClientInfo(clients);
        }

        private void ShowCurrentClientInfo(List<Client> clients)
        {
            if (isDeposit)
            {

                accountNumberLabel.Content = $"Номер счета: {clients[clientId].DepositAccountClient.AccountNumber.ToString}";
                string currentBalance = clients[clientId].DepositAccountClient.Balance.ToString("0.00");
                string newBalance = clients[clientId].CheckAccumulation(isDeposit).ToString("0.00");
                balanceLabel.Content = $"Баланс: {currentBalance} (через год: {newBalance})";
                string status = clients[clientId].DepositAccountClient.IsOpen ? "Открыт" : "Закрыт";
                statusLabel.Content = $"Статус счета: {status}";
                if (clients[clientId].DepositAccountClient.IsOpen) actionAccountButton.Content = "Закрыть счет";
                else actionAccountButton.Content = "Открыть счет";
                clientStatusLabel.Content = clients[clientId].GetType().Name;
                changeLogListBox.ItemsSource = clients[clientId].DepositAccountClient.ChangeLog;
                changeLogListBox.Items.Refresh();
            }
            else 
            {

                accountNumberLabel.Content = $"Номер счета: {clients[clientId].AccountClient.AccountNumber.ToString}";
                string currentBalance = clients[clientId].AccountClient.Balance.ToString("0.00");
                string newBalance = clients[clientId].CheckAccumulation(isDeposit).ToString("0.00");
                balanceLabel.Content = $"Баланс: {currentBalance} (через год: {newBalance})";
                string status = clients[clientId].AccountClient.IsOpen ? "Открыт" : "Закрыт";
                statusLabel.Content = $"Статус счета: {status}";
                if (clients[clientId].AccountClient.IsOpen) actionAccountButton.Content = "Закрыть счет";
                else actionAccountButton.Content = "Открыть счет";
                clientStatusLabel.Content = clients[clientId].GetType().Name;
                changeLogListBox.ItemsSource = clients[clientId].AccountClient.ChangeLog;
                changeLogListBox.Items.Refresh();

                status.Split("ff");
            }
        }

        private void topUpButton_Click(object sender, RoutedEventArgs e)
        {
            if (double.TryParse(amountTextBox.Text, out double d))
            {
                balanceOperation = clients[clientId].TopUpBalance;
                balanceOperation(double.Parse(amountTextBox.Text), isDeposit);
                ShowCurrentClientInfo();
            }
            else
            {
                MessageBox.Show("Введите корректную сумму пополнения!");
            }
        }

        private void withdrawButton_Click(object sender, RoutedEventArgs e)
        {
            if (double.TryParse(amountTextBox.Text, out double d))
            {
                bool isSuccess = CheckAccounts(clientId);
                if (balanceOperation!=null)
                {
                    balanceOperation = clients[clientId].WithdrawBalance;
                    balanceOperation(d, isDeposit);
                    ShowCurrentClientInfo();
                }
            }
            else
            {
                MessageBox.Show("Введите корректную сумму вывода!");
            }
        }

        private void actionAccountButton_Click(object sender, RoutedEventArgs e)
        {
            bool isOpenDepositAccount = isDeposit && clients[clientId].DepositAccountClient.IsOpen;
            bool isOpenAccount = !isDeposit && clients[clientId].AccountClient.IsOpen;
            if (isOpenDepositAccount || isOpenAccount) simpleOperation = clients[clientId].CloseAccount;
            else simpleOperation = clients[clientId].OpenAccount;
            simpleOperation(isDeposit);
            ShowCurrentClientInfo();
        }

        private void transferButton_Click(object sender, RoutedEventArgs e)
        {
            try
            {
                if (string.IsNullOrEmpty(amountTextBox.Text))
                {
                    throw new TextBoxInputException("Введите сумму перевода!");
                }
                else
                {
                    double d = double.Parse(amountTextBox.Text);

                    int transferId = Convert.ToInt32(transferComboBox.SelectedValue);
                    int clientToId = clients.FindIndex(c => c.Id == transferId);
                    int fromId = Convert.ToInt32(clientsComboBox.SelectedValue);
                    if (fromId != transferId)
                    {
                        bool isSuccess = CheckAccounts(clientToId);
                        if (isSuccess)
                        {
                            transferOperation = clients[clientId].TransferBalance;
                            transferOperation(clients[clientToId], d, isDeposit);
                            ShowCurrentClientInfo();
                        }
                    }
                    else MessageBox.Show("Выберите другого клиетна!");
                }
            }
            catch (FormatException)
            {
                MessageBox.Show("Введите корректную сумму перевода!");
            }
            catch (TextBoxInputException ex)
            {
                MessageBox.Show(ex.Message);
            }
        }

        private bool CheckAccounts(int transferId)
        {
            if (isDeposit)
            {
                try
                {
                    if (!clients[transferId].DepositAccountClient.IsOpen || !clients[clientId].DepositAccountClient.IsOpen)
                    {
                        throw new AccountNotOpenException("У клиента не открыт счет!");
                    }
                    else if (clients[clientId].DepositAccountClient.Balance < double.Parse(amountTextBox.Text))
                    {
                        throw new AccountNotOpenException("У клиента недостаточно средств на счете!");
                    }
                }
                catch (AccountNotOpenException ex)
                {
                    MessageBox.Show(ex.Message);
                    return false;
                }
            }
            else
            {
                try
                {
                    if (!clients[transferId].AccountClient.IsOpen || !clients[clientId].AccountClient.IsOpen)
                    {
                        throw new AccountNotOpenException("У клиента не открыт счет!");
                    }
                    else if (clients[clientId].AccountClient.Balance < double.Parse(amountTextBox.Text))
                    {
                        throw new AccountNotOpenException("У клиента недостаточно средств на счете!");
                    }
                }
                catch (AccountNotOpenException ex)
                {
                    MessageBox.Show(ex.Message);
                    return false;
                }
            }
            return true;
        }

        private void SaveButton_Click(object sender, RoutedEventArgs e)
        {
            FileWork.SaveToFiles(clients);
        }

        private void Account_OnAccountChanged()
        {
            ShowCurrentClientInfo();
        }

        private void Account_OnTransfered(string msg)
        {
            MessageBox.Show(msg);
        }
    }
}
