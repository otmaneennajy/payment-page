<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Test Payment</title>
  <meta name="viewport" content="width=device-width,initial-scale=1">
</head>
<body style="font-family:Arial,sans-serif;padding:40px;max-width:360px;margin:auto">
  <h1>Test Payment</h1>

  <input id="phone" type="tel"    placeholder="Phone (e.g. 0712345678)" style="width:100%;padding:8px;margin:8px 0">
  <input id="amount" type="number" placeholder="Amount XOF"            style="width:100%;padding:8px;margin:8px 0">
  <button id="pay-btn" style="width:100%;padding:10px;background:#007acc;color:#fff;border:none;cursor:pointer">
    Pay Now
  </button>
  <p id="msg" style="color:red"></p>

  <script>
    // ———  Place your JWT here  ———
    var token = "        "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6Im90bWFuZW5hamk4QGdtYWlsLmNvbSIsInN1YiI6NzUsImlhdCI6MTc1MjY5MjUyMiwiZXhwIjoxNzUyNzEwNTIyfQ.NFvbdqE6-f9q1owDVWnHXA23CSddsu5hvFbvI1rcceA",
";
    console.log("Using token:", token);

    document.getElementById("pay-btn").addEventListener("click", async function(){
      var phone = document.getElementById("phone").value.trim();
      var amount = document.getElementById("amount").value.trim();
      var msg = document.getElementById("msg");
      msg.textContent = "";

      if (!phone || !amount) {
        msg.textContent = "Enter phone and amount.";
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
            amount: parseInt(amount,10),
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
