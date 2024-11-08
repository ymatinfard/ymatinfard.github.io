---
# the default layout is 'page'
icon: fas fa-info-circle
order: 5
---

<html lang="en">
<head>
    <meta charset="UTF-8">
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
        }
        .about-container {
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
            max-width: 600px;
            text-align: center;
        }
        h1 {
            font-size: 2em;
            color: #333;
        }
        p {
            font-size: 1.1em;
            color: #555;
            line-height: 1.6;
        }
        .highlight {
            color: #007ACC;
            font-weight: bold;
        }
        .contact-info {
            margin-top: 20px;
            font-size: 1em;
        }
        .contact-info a {
            color: #007ACC;
            text-decoration: none;
            font-weight: bold;
        }
        .contact-info a:hover {
            color: #005A9E;
        }
        .emoji {
            font-size: 1.5em;
            margin-right: 5px;
        }
        .footer {
            margin-top: 20px;
            font-style: italic;
            color: #888;
            font-size: 0.9em;
        }
      .greeting {
          font-size: 1.8em; 
          font-weight: bold; 
          color: #ff5733;
      }
    </style>
</head>
<body>

<div class="about-container">
<p><span class="emoji">👋</span> <span class="greeting">Ciao! I'm Yousef Matinfard</span></p>
    <p>Location: Italy <br>
       Email: <a href="mailto:matinfard.y@gmail.com">matinfard.y@gmail.com</a>
    </p>
    <p>I’m an Android developer by day, espresso enthusiast by night. When I’m not writing code, you can find me arguing with my phone about whether Kotlin is superior to Java (spoiler: Kotlin always wins). I moved to Italy for the pasta, but I stayed for the <em>pasta</em> and the chance to build cool Android apps surrounded by stunning views.</p>
    <p>I’ve got a knack for making apps run smoothly, navigating the endless world of coroutines, and turning pixel-perfect designs into reality. Writing clean, efficient code is my game, and I bring a little Italian flavor to every project I work on.</p>
    <div class="footer">
        *Best enjoyed with a cup of espresso.*
    </div>
</div>

</body>
</html>
