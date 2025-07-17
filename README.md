<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Mobile Payment</title>
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <style>
    body{font-family:Arial,sans-serif;padding:40px;max-width:360px;margin:auto;background:#f5f5f5}
    input,button{width:100%;padding:10px;margin:8px 0;border:1px solid #ccc;border-radius:5px;box-sizing:border-box}
    button{background:#007acc;color:#fff;border:none;cursor:pointer}
    button:hover{background:#005f99}
    .message{color:red;text-align:center;margin-top:10px}
  </style>
</head>
<body>

  <h1>Mobile Payment</h1>

  <input id="phone" type="tel"    placeholder="Phone (e.g. 0712345678)">
  <input id="amount" type="number" placeholder="Amount XOF">
  <button id="pay-btn">Pay Now</button>
  <p id="msg" class="message"></p>

  <script>
    // 1) هنا ضعُ JWT النظيف الذي حصلت عليه (ASCII فقط):
    var token = "        "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6Im90bWFuZW5hamk4QGdtYWlsLmNvbSIsInN1YiI6NzUsImlhdCI6MTc1MjY5MjUyMiwiZXhwIjoxNzUyNzEwNTIyfQ.NFvbdqE6-f9q1owDVWnHXA23CSddsu5hvFbvI1rcceA",
";

    document.getElementById("pay-btn").addEventListener("click", async function(){
      var phone  = document.getElementById("phone").value.trim();
      var amount = document.getElementById("amount").value.trim();
      var msg    = document.getElementById("msg");
      msg.textContent = "";

      if (!phone || !amount) {
        msg.textContent = "Please enter phone and amount.";
        return;
      }

      try {
        let res = await fetch("https://v2.b-pay.co/service/api/v1/paiement", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
            "Authorization": "Bearer " + token
          },
          body: JSON.stringify({
            payment_method: "SN_PM_OM",
            merchant_transaction_id: "TEST-" + Date.now(),
            amount: parseInt(amount, 10),
            telephone: phone,
            refercence_cl: "cli-" + Math.random().toString(36).substr(2,8),
            currency: "XOF",
            success_url: window.location.href,
            failed_url:  window.location.href,
            notify_url:  window.location.origin + "/callback"
          })
        });
        let data = await res.json();
        console.log("Response:", data);
        if (data.status === "SUCCESSFULL") {
          alert("Payment succeeded");
        } else {
          alert("Payment error: " + data.message);
        }
      } catch (e) {
        alert("Technical error: " + e.message);
      }
    });
  </script>

</body>
</html>
