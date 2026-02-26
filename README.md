<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Dixon Resultant & Polynomial System Solver</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 900px;
            margin: auto;
            padding: 40px;
            line-height: 1.6;
        }
        h1 {
            border-bottom: 2px solid #444;
            padding-bottom: 10px;
        }
        code {
            background: #f4f4f4;
            padding: 3px 6px;
            border-radius: 5px;
        }
        pre {
            background: #f4f4f4;
            padding: 15px;
            overflow-x: auto;
        }
        a {
            color: #0366d6;
        }
        .box {
            background: #f9f9f9;
            padding: 15px;
            border-left: 4px solid #0366d6;
            margin: 20px 0;
        }
    </style>
</head>
<body>

<h1>Dixon Resultant & Polynomial System Solver</h1>

<p>
A high-performance C implementation for computing Dixon resultants 
and solving polynomial systems over finite fields.
</p>

<div class="box">
<strong>GitHub Repository:</strong><br>
<a href="https://github.com/DixonRes/DixonRes">
https://github.com/DixonRes/DixonRes
</a>
</div>

<h2>Features</h2>
<ul>
<li>Dixon resultant computation for elimination</li>
<li>Polynomial system solver (n×n systems)</li>
<li>Triangular ideal reduction</li>
<li>Prime and extension finite fields</li>
<li>Command-line and file-based input</li>
</ul>

<h2>Build</h2>
<pre>
make
</pre>

<h2>Example</h2>
<pre>
./dixon --solve "x^2 + y^2 + z^2 - 6, x + y + z - 4, x*y*z - x - 1" 257
</pre>

<h2>Dependencies</h2>
<ul>
<li>FLINT (recommended 3.4.0)</li>
<li>PML (optional acceleration)</li>
</ul>

<h2>License</h2>
<p>GNU GPL v3.0</p>

</body>
</html>

