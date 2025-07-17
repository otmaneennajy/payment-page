<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Mobile Payment</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <!-- Google Font -->
  <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;600&display=swap" rel="stylesheet">
  <style>
    /* Base Reset */
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body {
      font-family: 'Poppins', sans-serif;
      background: linear-gradient(135deg, #74ABE2 0%, #5563DE 100%);
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
      color: #333;
    }
    .container {
      background: #fff;
      padding: 2rem;
      border-radius: 12px;
      width: 100%;
      max-width: 400px;
      box-shadow: 0 8px 24px rgba(0,0,0,0.1);
      position: relative;
      overflow: hidden;
    }
    .container::before {
      content: '';
      position: absolute;
      top: -50%;
      right: -50%;
      width: 200%;
      height: 200%;
      background: radial-gradient(circle at center, rgba(85,99,222,0.2), transparent 70%);
      transform: rotate(45deg);
    }
    h1 {
      text-align: center;
      margin-bottom: 1.5rem;
      font-weight: 600;
      color: #2c3e50;
    }
    label {
      display: block;
      margin-top: 1rem;
      font-size: 0.9rem;
      font-weight: 500;
    }
    input, select, button {
      width: 100%;
      padding: 0.8rem 1rem;
      margin-top: 0.5rem;
      font-size: 1rem;
      border-radius: 8px;
      border: 1px solid #ccc;
      outline: none;
      transition: border-color .3s, box-shadow .3s;
    }
    input:focus, select:focus {
      border-color: #5563DE;
      box-shadow: 0 0 0 3px rgba(85,99,222,0.2);
    }
    button {
      background: #5563DE;
      color: #fff;
      border: none;
      font-weight: 600;
      margin-top: 1.5rem;
      cursor: pointer;
      transition: background .3s, transform .2s;
    }
    button:hover {
      background: #2c3e50;
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
          message.textContent = 'Payment error: ' + payData.message;
        }
      } catch (err) {
        message.textContent = 'Technical error: ' + err.message;
      }
    });
  </script>
</body>
</html>
