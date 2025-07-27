import React, { useState, useEffect } from 'react';
import { View, Text, Button, StyleSheet, TextInput, ScrollView, Alert } from 'react-native';
import { WebView } from 'react-native-webview';

export default function App() {
  const [balance, setBalance] = useState(10000);
  const [username, setUsername] = useState('');
  const [loggedIn, setLoggedIn] = useState(false);
  const [prices, setPrices] = useState({});
  const [refCode] = useState(() => Math.random().toString(36).substring(2, 8).toUpperCase());
  const [invites, setInvites] = useState(0);
  const [transactions, setTransactions] = useState([]);

  useEffect(() => {
    if (loggedIn) {
      fetchPrices();
      const interval = setInterval(fetchPrices, 10000);
      return () => clearInterval(interval);
    }
  }, [loggedIn]);

  const fetchPrices = async () => {
    try {
      const res = await fetch('https://api.exchangerate.host/latest?base=USD&symbols=EUR,GBP,ZAR,XAU,BTC');
      const data = await res.json();
      setPrices(data.rates);
    } catch (err) {
      console.error('Price fetch failed:', err);
    }
  };

  const copyReferral = () => {
    Alert.alert('Referral Code', `Your code: ${refCode}\nCopied to clipboard (simulated).`);
  };

  const simulateInvite = () => {
    const reward = 50;
    setInvites(invites + 1);
    setBalance(balance + reward);
    setTransactions(prev => [...prev, { id: Date.now(), type: 'Referral Bonus', amount: reward }]);
  };

  const handleDeposit = () => {
    const amount = 1000;
    setBalance(balance + amount);
    setTransactions(prev => [...prev, { id: Date.now(), type: 'Deposit', amount }]);
  };

  const handleWithdraw = () => {
    const amount = 500;
    if (balance >= amount) {
      setBalance(balance - amount);
      setTransactions(prev => [...prev, { id: Date.now(), type: 'Withdrawal', amount: -amount }]);
    } else {
      Alert.alert('Not enough balance', 'Cannot withdraw more than available.');
    }
  };

  return (
    <ScrollView contentContainerStyle={styles.container}>
      {!loggedIn ? (
        <>
          <Text style={styles.title}>Welcome to MT Market</Text>
          <TextInput
            placeholder="Enter Username"
            style={styles.input}
            onChangeText={setUsername}
          />
          <Button title="Login" onPress={() => setLoggedIn(true)} />
        </>
      ) : (
        <>
          <Text style={styles.welcome}>Hello {username}</Text>
          <Text style={styles.balance}>Balance: ${balance}</Text>

          <Button title="Deposit $1000" onPress={handleDeposit} />
          <Button title="Withdraw $500" onPress={handleWithdraw} />

          <Text style={styles.section}>📊 Market Prices</Text>
          {Object.keys(prices).length === 0 ? (
            <Text>Loading prices...</Text>
          ) : (
            <>
              <Text>EUR/USD: {(1 / prices['EUR']).toFixed(4)}</Text>
              <Text>GBP/USD: {(1 / prices['GBP']).toFixed(4)}</Text>
              <Text>ZAR/USD: {(1 / prices['ZAR']).toFixed(4)}</Text>
              <Text>XAU/USD: {(1 / prices['XAU']).toFixed(2)}</Text>
              <Text>BTC/USD: {(1 / prices['BTC']).toFixed(2)}</Text>
            </>
          )}

          <Text style={styles.section}>👥 Referral System</Text>
          <Text>Your Code: {refCode}</Text>
          <Button title="Copy Code" onPress={copyReferral} />
          <Button title="Simulate Invite" onPress={simulateInvite} />
          <Text>Referrals: {invites} | Earned: ${invites * 50}</Text>

          <Text style={styles.section}>💳 Transactions</Text>
          {transactions.length === 0 ? (
            <Text>No transactions yet.</Text>
          ) : (
            transactions
              .slice()
              .reverse()
              .map((t) => (
                <Text key={t.id}>
                  {t.type}: ${t.amount > 0 ? '+' : ''}{t.amount}
                </Text>
              ))
          )}

          <Text style={styles.section}>📈 Live Chart</Text>
          <View style={{ height: 400 }}>
            <WebView
              source={{
                uri: 'https://s.tradingview.com/widgetembed/?frameElementId=tradingview_eurusd&symbol=FX:EURUSD&interval=30&hidesidetoolbar=1&symboledit=1&saveimage=1&toolbarbg=f1f3f6&studies=[]&theme=light&style=1&timezone=Africa/Johannesburg'
              }}
              style={{ flex: 1 }}
            />
          </View>
        </>
      )}
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  container: { flexGrow: 1, padding: 20, backgroundColor: '#fff' },
  title: { fontSize: 28, fontWeight: 'bold', textAlign: 'center', marginBottom: 20 },
  welcome: { fontSize: 24, textAlign: 'center', marginVertical: 10 },
  balance: { fontSize: 20, textAlign: 'center', marginBottom: 20 },
  input: {
    borderWidth: 1,
    borderColor: '#ccc',
    borderRadius: 8,
    padding: 10,
    marginBottom: 10
  },
  section: {
    fontSize: 22,
    fontWeight: 'bold',
    marginTop: 30,
    marginBottom: 10
  }
});
