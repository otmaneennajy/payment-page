<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Mobile Payment</title>
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <style>
    /* Reset */
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      font-family: system-ui, sans-serif;
      background: linear-gradient(135deg, #89f7fe 0%, #66a6ff 100%);
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
      color: #333;
    }

    .form-container {
      width: 100%;
      max-width: 360px;
      padding: 2rem;
      border-radius: 16px;
      background: rgba(255,255,255,0.15);
      box-shadow: 0 8px 32px rgba(0,0,0,0.1);
      backdrop-filter: blur(10px);
      border: 1px solid rgba(255,255,255,0.3);
      color: #222;
    }

    h1 {
      text-align: center;
      margin-bottom: 1.5rem;
      font-size: 1.4rem;
      font-weight: 600;
    }

    label {
      display: block;
      margin-top: 1rem;
      font-size: 0.9rem;
    }

    input, select, button {
      width: 100%;
      padding: 0.75rem 1rem;
      margin-top: 0.5rem;
      font-size: 1rem;
      border-radius: 8px;
      border: none;
      background: rgba(255,255,255,0.4);
      color: #222;
      outline: none;
      backdrop-filter: blur(5px);
      transition: background .3s, transform .2s;
    }

    input:focus, select:focus {
      background: rgba(255,255,255,0.6);
    }

    button {
      background: rgba(255,255,255,0.5);
      cursor: pointer;
      margin-top: 1.5rem;
      font-weight: 600;
    }

    button:hover {
      background: rgba(255,255,255,0.7);
      transform: translateY(-2px);
    }

    .message {
      margin-top: 1rem;
      text-align: center;
      font-size: 0.9rem;
      color: #e74c3c;
      min-height: 1.2em;
    }
  </style>
</head>
<body>
  <div class="form-container">
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
    document.getElementById('pay-btn').addEventListener('click', async () => {
      const phone   = document.getElementById('phone').value.trim();
      const amount  = document.getElementById('amount').value.trim();
      const method  = document.getElementById('method').value;
      const msgEl   = document.getElementById('message');
      msgEl.textContent = '';

      if (!phone || !amount) {
        msgEl.textContent = 'Please enter both phone number and amount.';
        return;
      }

      try {
        // 1) Login to get token
        const loginRes = await fetch('https://v2.b-pay.co/service/api/v1/oauth/login', {
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

        // 2) Make payment
        const payRes = await fetch('https://v2.b-pay.co/service/api/v1/paiement', {
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
          msgEl.textContent = 'Payment error: ' + payData.message;
        }
      } catch (err) {
        msgEl.textContent = 'Technical error: ' + err.message;
      }
    });
  </script>
</body>
</html>
