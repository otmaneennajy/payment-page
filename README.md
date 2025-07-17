<!doctype html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Mobile Payment</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f5f5f5;
      margin: 0;
      padding: 20px;
    }
    .container {
      max-width: 360px;
      margin: 40px auto;
      background: #fff;
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    }
    h1 {
      margin-bottom: 20px;
      font-size: 1.4rem;
      text-align: center;
    }
    label {
      display: block;
      margin-top: 15px;
      font-size: 0.9rem;
    }
    input, select, button {
      width: 100%;
      padding: 10px;
      margin-top: 8px;
      font-size: 1rem;
      border: 1px solid #ccc;
      border-radius: 5px;
      box-sizing: border-box;
    }
    button {
      background: #007acc;
      color: #fff;
      border: none;
      cursor: pointer;
      margin-top: 20px;
    }
    button:hover {
      background: #005f99;
    }
    .message {
      margin-top: 15px;
      text-align: center;
      font-size: 0.9rem;
      color: #d00;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>Mobile Payment</h1>

    <label for="phone">Phone Number</label>
    <input id="phone" type="tel" placeholder="e.g. 0712345678">

    <label for="amount">Amount (XOF)</label>
    <input id="amount" type="number" placeholder="e.g. 5000">

    <label for="method">Payment Method</label>
    <select id="method">
      <option value="SN_PM_OM">Orange Money</option>
      <option value="SN_PM_WAVE">Wave</option>
    </select>

    <button id="pay-btn">Pay Now</button>
    <p class="message" id="message"></p>
  </div>

  <script>
    // base URL of your Vercel proxy
    const proxyBase = 'https://bpay-proxy-qenzas2yh-otmanes-projects-d2b10120.vercel.app';

    document.getElementById('pay-btn').addEventListener('click', async () => {
      const phone   = document.getElementById('phone').value.trim();
      const amount  = document.getElementById('amount').value.trim();
      const method  = document.getElementById('method').value;
      const message = document.getElementById('message');
      message.textContent = '';

      if (!phone || !amount) {
        message.textContent = 'Please enter both phone number and amount.';
        return;
      }

      try {
        // 1) Login to get token via proxy
        const loginRes = await fetch(proxyBase + '/api/login-proxy', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            email: 'otmanenaji8@gmail.com',
            password: 'qAiex3d'
          })
        });
        const loginData = await loginRes.json();
        if (!loginData.authorisation || !loginData.authorisation.token) {
          throw new Error(loginData.message || 'Login failed');
        }
        const token = loginData.authorisation.token;

        // 2) Make payment via proxy
        const payRes = await fetch(proxyBase + '/api/pay-proxy', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': 'Bearer ' + token
          },
          body: JSON.stringify({
            payment_method: method,
            merchant_transaction_id: `STD-PAY-${Date.now()}`,
            amount: parseInt(amount, 10),
            telephone: phone,
            refercence_cl: `cli-${Math.random().toString(36).substr(2,8)}`,
            currency: 'XOF',
            success_url: window.location.origin + '/success.html',
            failed_url:  window.location.origin + '/failed.html',
            notify_url:  window.location.origin + '/callback'
          })
        });
        const payData = await payRes.json();

        if (payData.status === 'SUCCESSFULL') {
          window.location.href = payData.redirect_url || 'success.html';
        } else {
          message.textContent = 'Payment error: ' + payData.message;
        }
      } catch (err) {
        message.textContent = 'Technical error: ' + err.message;
      }
    });
  </script>
</body>
</html>
