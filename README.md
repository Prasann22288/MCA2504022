1. Setup once
Install Node.js LTS, then check `node -v` and `npm -v`. For each question create a folder, open Terminal in it, run `npm init -y`,
save the code, then run the shown command. Browser examples use `http://localhost:3000`.
npm init -y
npm install express
# For MySQL questions:
npm install mysql2
# For MongoDB/Mongoose questions:
npm install mongoose
# For file uploads:
npm install multer
# For Lodash:
npm install lodash
Use only the packages needed for that question. Start a local database service before running database examples. Never
expose real database passwords in shared code.
2. Browser and form pattern
For questions that display results in a browser, Express serves the page. For POST forms, the server parses URL-encoded form
data and returns the submitted values safely escaped. This helper is reused in the relevant examples.
// Save as server.js
const express = require('express');
const app = express();
app.use(express.urlencoded({ extended: false }));
app.get('/', (req, res) => res.send('Server is running'));
app.listen(3000, () => console.log('http://localhost:3000'));
3. Common questions (1–9)
1. User-defined module – read student.json and display it in
browser
// student.json
{"rollNo": 101, "name": "Asha", "course": "BCA", "marks": 88}
// server.js
const express = require('express');
const fs = require('fs');
const path = require('path');
const app = express();
app.get('/', (req, res) => {
 try {
 const raw = fs.readFileSync(path.join(__dirname, 'student.json'), 'utf8');
 const student = JSON.parse(raw);
 res.type('html').send('<h1>Student Details</h1><pre>' +
 escapeHtml(JSON.stringify(student, null, 2)) + '</pre>');
Node.js Practical Question Bank — Solutions Page 2
 } catch (e) { res.status(500).send('Could not read/parse student.json: ' + e.message); }
});
function escapeHtml(s) { return s.replace(/[&<>"]/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;'}[c])); }
app.listen(3000, () => console.log('Open http://localhost:3000'));
Run: npm install express; node server.js
2. Check whether a path is a file or directory; list directory
contents
// check.js
const fs = require('fs');
const path = require('path');
const target = process.argv[2];
if (!target) { console.log('Usage: node check.js <path>'); process.exit(1); }
const p = path.resolve(target);
try {
 const stat = fs.statSync(p);
 if (stat.isFile()) console.log('It is a file:', p);
 else if (stat.isDirectory()) {
 console.log('It is a directory:', p);
 console.log(fs.readdirSync(p).join('\n') || '(empty directory)');
 } else console.log('It is neither a regular file nor a directory.');
} catch (e) { console.error('Path not found or inaccessible:', e.message); }
Run: node check.js .
3. Handle user login and logout with EventEmitter
// events.js
const EventEmitter = require('events');
const userEvents = new EventEmitter();
userEvents.on('login', user => console.log(`${user} logged in`));
userEvents.on('logout', user => console.log(`${user} logged out`));
userEvents.emit('login', 'Prasann');
userEvents.emit('logout', 'Prasann');
Run: node events.js
4. Express routing: Home, About, Contact Us
// server.js
const express = require('express');
const app = express();
app.get('/', (req, res) => res.send('<h1>Home</h1>'));
app.get('/about', (req, res) => res.send('<h1>About Us</h1>'));
app.get('/contact', (req, res) => res.send('<h1>Contact Us</h1><p>contact@example.com</p>'));
app.listen(3000, () => console.log('Open http://localhost:3000'));
Run: npm install express; node server.js
5. Accept employee details using HTML form and POST
// server.js
const express = require('express');
const app = express();
app.use(express.urlencoded({ extended: false }));
const esc = s => String(s || '').replace(/[&<>"]/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;'}[c]));
app.get('/', (req, res) => res.send(`<h1>Employee Form</h1><form method="post" action="/employee">
EID: <input name="eid" required><br>EName: <input name="ename" required><br>
Designation: <input name="designation" required><br>Company: <input name="company" required><br>
<button>Submit</button></form>`));
app.post('/employee', (req, res) => {
 const {eid, ename, designation, company} = req.body;
 res.send(`<h1>Employee Details</h1><p>EID: ${esc(eid)}</p><p>Name: ${esc(ename)}</p><p>Designation: ${esc(designation)}</p><p>Company});
app.listen(3000, () => console.log('Open http://localhost:3000'));
Run: npm install express; node server.js
6. Insert five book documents into MongoDB
// insertBooks.js
const mongoose = require('mongoose');
const Book = mongoose.model('Book', new mongoose.Schema({
 title: String, author: String, year: Number, price: Number
}, { collection: 'books' }));
async function main() {
Node.js Practical Question Bank — Solutions Page 3
 await mongoose.connect('mongodb://127.0.0.1:27017/library');
 await Book.insertMany([
 {title:'Node Basics',author:'A. Rao',year:2022,price:450},
 {title:'JavaScript Guide',author:'B. Shah',year:2023,price:650},
 {title:'Web Development',author:'C. Das',year:2021,price:700},
 {title:'Express in Practice',author:'D. Patil',year:2024,price:550},
 {title:'MongoDB Intro',author:'E. Joshi',year:2020,price:390}
 ]);
 console.log('Inserted 5 books');
 await mongoose.disconnect();
}
main().catch(async e => { console.error(e.message); await mongoose.disconnect(); });
Run: npm install mongoose; node insertBooks.js
7. Fetch books where price > 500 from MongoDB
// expensiveBooks.js
const mongoose = require('mongoose');
const Book = mongoose.model('Book', new mongoose.Schema({
 title:String, author:String, year:Number, price:Number
}, {collection:'books'}));
async function main() {
 await mongoose.connect('mongodb://127.0.0.1:27017/library');
 const books = await Book.find({price: {$gt: 500}}).lean();
 console.table(books);
 await mongoose.disconnect();
}
main().catch(e => { console.error(e.message); process.exitCode=1; });
Run: npm install mongoose; node expensiveBooks.js
8. Student CRUD operations using MySQL
-- Run once in MySQL
CREATE DATABASE IF NOT EXISTS college_db;
USE college_db;
CREATE TABLE IF NOT EXISTS students (
 id INT PRIMARY KEY, name VARCHAR(100), course VARCHAR(50), marks INT
);
-- Save below as studentCrud.js
const mysql = require('mysql2/promise');
async function main() {
 const db = await mysql.createConnection({host:'localhost', user:'root', password:'YOUR_PASSWORD', database:'college_db'});
 await db.execute('INSERT INTO students(id,name,course,marks) VALUES(?,?,?,?) ON DUPLICATE KEY UPDATE name=VALUES(name)', [1,'Asha','B console.log('All:', (await db.query('SELECT * FROM students'))[0]);
 await db.execute('UPDATE students SET marks=? WHERE id=?', [92,1]);
 await db.execute('DELETE FROM students WHERE id=?', [1]);
 await db.end();
}
main().catch(e => console.error(e.message));
Run: npm install mysql2; set YOUR_PASSWORD, then node studentCrud.js
9. Upload a text file
// server.js
const express = require('express');
const multer = require('multer');
const path = require('path');
const fs = require('fs');
const app = express();
fs.mkdirSync('uploads', {recursive:true});
const upload = multer({dest:'uploads/', limits:{fileSize:2*1024*1024}});
app.get('/', (req,res) => res.send(`<h1>Upload text file</h1><form method="post" enctype="multipart/form-data" action="/upload"><inputapp.post('/upload', upload.single('file'), (req,res) => {
 if (!req.file) return res.status(400).send('Choose a file');
 const ext = path.extname(req.file.originalname).toLowerCase();
 if (ext !== '.txt') { fs.unlinkSync(req.file.path); return res.status(400).send('Only .txt files allowed'); }
 res.send('Uploaded: ' + path.basename(req.file.originalname));
});
app.listen(3000, () => console.log('Open http://localhost:3000'));
Run: npm install express multer; node server.js
4. Questions only in the first list (10–24)
10. Sum of digits using a user-defined module
// digitSum.js
Node.js Practical Question Bank — Solutions Page 4
exports.sumDigits = n => String(Math.abs(Math.trunc(Number(n)))).split('').reduce((s,d)=>s+Number(d),0);
// main.js
const {sumDigits} = require('./digitSum');
const n = Number(process.argv[2]);
if (!Number.isFinite(n)) throw new Error('Usage: node main.js <number>');
console.log('Sum of digits =', sumDigits(n));
Run: node main.js 12345
11. Display profile on a webpage
// server.js
const express = require('express');
const app = express();
app.get('/', (req,res) => res.send(`<h1>My Profile</h1><p>Name: Your Name</p><p>Course: MCA</p><p>Skills: Node.js, JavaScript, MySQL</app.listen(3000, () => console.log('Open http://localhost:3000'));
Run: npm install express; node server.js
12. List files in a directory
// list.js
const fs = require('fs');
const dir = process.argv[2] || '.';
try { fs.readdirSync(dir, {withFileTypes:true}).forEach(d => console.log((d.isDirectory()?'[DIR] ':'[FILE] ')+d.name)); }
catch(e) { console.error(e.message); }
Run: node list.js .
13. Insert three student records into MySQL
-- SQL setup
CREATE DATABASE IF NOT EXISTS college_db;
USE college_db;
CREATE TABLE IF NOT EXISTS students (id INT PRIMARY KEY, name VARCHAR(80), course VARCHAR(50), marks INT);
-- insertStudents.js
const mysql = require('mysql2/promise');
async function main(){
 const db=await mysql.createConnection({host:'localhost',user:'root',password:'YOUR_PASSWORD',database:'college_db'});
 await db.execute('INSERT INTO students(id,name,course,marks) VALUES ?',
 [[1,'Asha','BCA',88],[2,'Ravi','MCA',81],[3,'Meera','BCA',91]]);
 console.log('Inserted 3 records'); await db.end();
}
main().catch(e=>console.error(e.message));
Run: npm install mysql2; node insertStudents.js
14. File read/write operations with custom events
// fileEvents.js
const fs = require('fs');
const {EventEmitter} = require('events');
const bus = new EventEmitter();
bus.on('write', ({file,text}) => fs.writeFile(file,text,e => {
 if(e) return console.error(e.message);
 console.log('Write complete'); bus.emit('read', file);
}));
bus.on('read', file => fs.readFile(file,'utf8',(e,data) => e ? console.error(e.message) : console.log('File:',data)));
bus.emit('write', {file:'notes.txt', text:'Hello from Node.js events'});
Run: node fileEvents.js
15. Validate username and password using MySQL
-- SQL: CREATE DATABASE IF NOT EXISTS college_db; USE college_db;
-- CREATE TABLE users (id INT AUTO_INCREMENT PRIMARY KEY, username VARCHAR(80) UNIQUE, password_hash VARCHAR(255));
-- Create a user with a bcrypt hash separately; never store plain passwords.
const mysql = require('mysql2/promise');
const bcrypt = require('bcryptjs');
async function main(){
 const [username,password] = process.argv.slice(2);
 const db=await mysql.createConnection({host:'localhost',user:'root',password:'YOUR_PASSWORD',database:'college_db'});
 const [rows]=await db.execute('SELECT password_hash FROM users WHERE username=?',[username]);
 const ok=rows.length===1 && await bcrypt.compare(password || '', rows[0].password_hash);
 console.log(ok?'Login successful':'Invalid username or password');
 await db.end();
}
main().catch(e=>console.error(e.message));
Run: npm install mysql2 bcryptjs; node login.js username password; table must contain a bcrypt hash
Node.js Practical Question Bank — Solutions Page 5
16. Array operations using Lodash
const _ = require('lodash');
const nums = [8, 2, 10, 4, 2];
console.log('Sorted:', _.sortBy(nums));
console.log('Maximum:', _.max(nums));
console.log('Reversed copy:', _.reverse([...nums]));
Run: npm install lodash; node array.js
17. Student objects using Lodash
const _ = require('lodash');
const students = [
 {rollNo:3,name:'Ravi',course:'MCA'},
 {rollNo:1,name:'Asha',course:'BCA'},
 {rollNo:2,name:'Meera',course:'BCA'}
];
console.log('Name-wise:', _.sortBy(students,['name']));
console.log('Roll no 2:', _.find(students,{rollNo:2}));
console.log('BCA only:', _.filter(students,{course:'BCA'}));
Run: npm install lodash; node students.js
18. Multiple events and callbacks
const {EventEmitter} = require('events');
const appEvents = new EventEmitter();
appEvents.on('start', () => console.log('Application started'));
appEvents.on('data', value => console.log('Data received:', value));
appEvents.on('stop', () => console.log('Application stopped'));
appEvents.emit('start'); appEvents.emit('data', 42); appEvents.emit('stop');
Run: node multiEvents.js
19. Voting eligibility using query parameters
const express = require('express');
const app = express();
app.get('/', (req,res) => res.send('Try /vote?name=Ravi&age=20'));
app.get('/vote', (req,res) => {
 const name=String(req.query.name||'Guest');
 const age=Number(req.query.age);
 if (!Number.isFinite(age) || req.query.age === undefined) return res.status(400).send('Provide a valid age');
 res.send(`${name.replace(/[&<>]/g,'')} is ${age>=18?'eligible':'not eligible'} to vote.`);
});
app.listen(3000,()=>console.log('Open http://localhost:3000'));
Run: npm install express; node vote.js
20. Copy file contents using fs
const fs = require('fs');
fs.copyFile('source.txt','destination.txt', err => {
 if(err) return console.error('Copy failed:',err.message);
 console.log('File copied successfully');
});
Run: Create source.txt first, then node copy.js
21. Student registration form validation
const express=require('express'); const app=express();
app.use(express.urlencoded({extended:false}));
app.get('/',(req,res)=>res.send(`<form method="post"><input name="name" placeholder="Name" required minlength="2"><input name="email" app.post('/',(req,res)=>{
 const {name,email,age}=req.body;
 if(!name||name.trim().length<2||!/^\S+@\S+\.\S+$/.test(email)||!Number.isInteger(Number(age))||Number(age)<16) return res.status(400) res.send('Registration valid for '+name.replace(/[&<>]/g,''));
});
app.listen(3000,()=>console.log('Open http://localhost:3000'));
Run: npm install express; node registration.js
22. Greet users according to current time using an event
const {EventEmitter}=require('events');
Node.js Practical Question Bank — Solutions Page 6
const clock=new EventEmitter();
clock.on('greet', name=>{
 const h=new Date().getHours();
 const greeting=h<12?'Good morning':h<17?'Good afternoon':'Good evening';
 console.log(`${greeting}, ${name}!`);
});
clock.emit('greet','Student');
Run: node greet.js
23. REST API using Express and MySQL
const express=require('express'); const mysql=require('mysql2/promise');
const app=express(); app.use(express.json());
const pool=mysql.createPool({host:'localhost',user:'root',password:'YOUR_PASSWORD',database:'college_db'});
app.get('/students',async(req,res)=>{try{const [r]=await pool.query('SELECT * FROM students');res.json(r)}catch(e){res.status(500).jsoapp.post('/students',async(req,res)=>{try{const {id,name,course,marks}=req.body;await pool.execute('INSERT INTO students(id,name,coursapp.put('/students/:id',async(req,res)=>{try{const {name,course,marks}=req.body;const [r]=await pool.execute('UPDATE students SET nameapp.delete('/students/:id',async(req,res)=>{try{const [r]=await pool.execute('DELETE FROM students WHERE id=?',[req.params.id]);res.jsapp.listen(3000,()=>console.log('REST API on port 3000'));
Run: Create college_db.students as in Q8; npm install express mysql2; node api.js
24. REST API using Express and MongoDB/Mongoose
const express=require('express'); const mongoose=require('mongoose');
const app=express(); app.use(express.json());
const Book=mongoose.model('Book',new mongoose.Schema({title:String,author:String,year:Number,price:Number},{collection:'books'}));
app.get('/books',async(req,res)=>{try{res.json(await Book.find())}catch(e){res.status(500).json({error:e.message})}});
app.post('/books',async(req,res)=>{try{res.status(201).json(await Book.create(req.body))}catch(e){res.status(400).json({error:e.messagapp.put('/books/:id',async(req,res)=>{try{const b=await Book.findByIdAndUpdate(req.params.id,req.body,{new:true,runValidators:true});iapp.delete('/books/:id',async(req,res)=>{try{const b=await Book.findByIdAndDelete(req.params.id);if(!b)return res.sendStatus(404);res.mongoose.connect('mongodb://127.0.0.1:27017/library').then(()=>app.listen(3000,()=>console.log('Books API on port 3000'))).catch(consoRun: npm install express mongoose; node booksApi.js
5. Questions only in the second list (25–38)
25. Prime and perfect number using a custom module
// numberCheck.js
exports.isPrime = n => Number.isInteger(n) && n>1 && !Array.from({length:Math.max(0,Math.floor(Math.sqrt(n))-1)},(_,i)=>i+2).some(d=>nexports.isPerfect = n => Number.isInteger(n) && n>1 && Array.from({length:n-1},(_,i)=>i+1).filter(d=>n%d===0).reduce((a,b)=>a+b,0)===n// main.js
const {isPrime,isPerfect}=require('./numberCheck');
const n=Number(process.argv[2]);
if(!Number.isInteger(n)||n<1) throw new Error('Usage: node main.js <positive integer>');
console.log(n,'prime?',isPrime(n)); console.log(n,'perfect?',isPerfect(n));
Run: node main.js 28
26. Area and circumference of a circle module
// circle.js
exports.area = r => Math.PI*r*r;
exports.circumference = r => 2*Math.PI*r;
// main.js
const c=require('./circle'); const r=Number(process.argv[2]);
if(!Number.isFinite(r)||r<0) throw new Error('Usage: node main.js <radius>=0');
console.log('Area:',c.area(r).toFixed(2));
console.log('Circumference:',c.circumference(r).toFixed(2));
Run: node main.js 7
27. Factorial using a custom module
// factorial.js
exports.factorial = n => {
 if(!Number.isInteger(n)||n<0||n>170) throw new Error('Use an integer from 0 to 170');
 let result=1; for(let i=2;i<=n;i++) result*=i; return result;
};
// main.js
const {factorial}=require('./factorial');
const n=Number(process.argv[2]); console.log(`${n}! = ${factorial(n)}`);
Run: node main.js 5
Node.js Practical Question Bank — Solutions Page 7
28. Create, write, append, read and delete a text file
const fs=require('fs/promises');
async function main(){
 const file='sample.txt';
 await fs.writeFile(file,'First line\n'); console.log('Created and wrote file');
 await fs.appendFile(file,'Appended line\n'); console.log('Appended');
 console.log('Contents:\n'+await fs.readFile(file,'utf8'));
 await fs.unlink(file); console.log('Deleted file');
}
main().catch(console.error);
Run: node fileOps.js
29. Copy source.txt to destination.txt using streams
const fs=require('fs');
const input=fs.createReadStream('source.txt');
const output=fs.createWriteStream('destination.txt');
input.on('error',e=>console.error('Read error:',e.message));
output.on('error',e=>console.error('Write error:',e.message));
output.on('finish',()=>console.log('Stream copy complete'));
input.pipe(output);
Run: Put content in source.txt, then node streamCopy.js
30. Add a student record to students.json
const fs=require('fs');
const file='students.json';
const student={rollNo:104,name:'Kiran',course:'MCA',fees:65000};
let students=[];
try { students=JSON.parse(fs.readFileSync(file,'utf8')); if(!Array.isArray(students)) throw new Error('JSON must be an array'); }
catch(e){ if(e.code!=='ENOENT') throw e; }
students.push(student);
fs.writeFileSync(file,JSON.stringify(students,null,2));
console.log('Student added');
Run: Start with students.json containing [] or let the program create it; node addStudent.js
31. orderPlaced event passes order ID and amount
const {EventEmitter}=require('events');
const orders=new EventEmitter();
orders.on('orderPlaced',({orderId,amount})=>console.log(`Order ${orderId} placed. Amount: ■${amount}`));
orders.emit('orderPlaced',{orderId:'ORD101',amount:1299});
Run: node order.js
32. Multiple listeners and remove listeners for cultural program
const {EventEmitter}=require('events');
const college=new EventEmitter();
function announce(){console.log('Annual cultural festival announced');}
function dance(){console.log('Dance competition scheduled');}
function music(){console.log('Music competition scheduled');}
college.on('annualProgram',announce);
college.on('annualProgram',dance);
college.on('annualProgram',music);
console.log('Listeners:',college.listenerCount('annualProgram'));
college.emit('annualProgram');
college.removeListener('annualProgram',dance);
console.log('After removing dance listener:',college.listenerCount('annualProgram'));
college.emit('annualProgram');
Run: node cultural.js
33. Accept student details using POST
const express=require('express');const app=express();app.use(express.urlencoded({extended:false}));
const esc=s=>String(s||'').replace(/[&<>"]/g,c=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;'}[c]));
app.get('/',(req,res)=>res.send(`<h1>Student Form</h1><form method="post" action="/student">Roll No <input name="rollno" required><br>app.post('/student',(req,res)=>{const {rollno,name,course,fees}=req.body;res.send(`<h1>Student Details</h1><p>Roll No: ${esc(rollno)}<app.listen(3000,()=>console.log('Open http://localhost:3000'));
Run: npm install express; node studentForm.js
Node.js Practical Question Bank — Solutions Page 8
34. Update employee salary by 10% using employee ID
-- SQL: CREATE TABLE Employee (eid INT PRIMARY KEY, ename VARCHAR(80), salary DECIMAL(10,2));
-- updateSalary.js
const mysql=require('mysql2/promise');
async function main(){
 const id=Number(process.argv[2]); if(!Number.isInteger(id)) throw new Error('Usage: node updateSalary.js <employeeId>');
 const db=await mysql.createConnection({host:'localhost',user:'root',password:'YOUR_PASSWORD',database:'company_db'});
 await db.execute('UPDATE Employee SET salary=ROUND(salary*1.10,2) WHERE eid=?',[id]);
 const [rows]=await db.execute('SELECT * FROM Employee WHERE eid=?',[id]);
 console.table(rows); await db.end();
}
main().catch(e=>console.error(e.message));
Run: Create company_db and Employee table; npm install mysql2; node updateSalary.js 1
35. Upload an image file
const express=require('express'),multer=require('multer'),fs=require('fs');
const app=express();fs.mkdirSync('uploads',{recursive:true});
const upload=multer({dest:'uploads/',limits:{fileSize:5*1024*1024},fileFilter:(req,file,cb)=>cb(null,/^image\/(png|jpeg|gif|webp)$/.teapp.get('/',(req,res)=>res.send(`<h1>Image upload</h1><form method="post" enctype="multipart/form-data" action="/upload"><input type="app.post('/upload',upload.single('image'),(req,res)=>{if(!req.file)return res.status(400).send('Select a valid image');res.send('Imageapp.listen(3000,()=>console.log('Open http://localhost:3000'));
Run: npm install express multer; node server.js
36. Fetch MySQL Student records and display in HTML table
const express=require('express'),mysql=require('mysql2/promise');const app=express();
const pool=mysql.createPool({host:'localhost',user:'root',password:'YOUR_PASSWORD',database:'college_db'});
const esc=s=>String(s??'').replace(/[&<>"]/g,c=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;'}[c]));
app.get('/',async(req,res)=>{try{const [rows]=await pool.query('SELECT * FROM students');res.send('<h1>Students</h1><table border="1">app.listen(3000,()=>console.log('Open http://localhost:3000'));
Run: Use college_db.students schema from Q8; npm install express mysql2; node table.js
37. Display all student records using MySQL
const mysql=require('mysql2/promise');
async function main(){
 const db=await mysql.createConnection({host:'localhost',user:'root',password:'YOUR_PASSWORD',database:'college_db'});
 const [rows]=await db.query('SELECT * FROM students'); console.table(rows); await db.end();
}
main().catch(e=>console.error(e.message));
Run: npm install mysql2; node showStudents.js
38. Display university programs on a webpage
const express=require('express');const app=express();
const programs=['BCA','MCA','BBA','MBA','B.Com','M.Com'];
app.get('/',(req,res)=>res.send('<h1>University Programs</h1><ul>'+programs.map(p=>'<li>'+p+'</li>').join('')+'</ul>'));
app.listen(3000,()=>console.log('Open http://localhost:3000'));
Run: npm install express; node programs.js
6. Appendix: practical exam checklist
• Save each example using the filename shown in its code comments or in the question heading.
• Run `npm install` for the packages listed for that question. The examples use CommonJS (`require`), so keep the default
package type (do not set `type: module`).
• For Express questions, keep the terminal open and visit `http://localhost:3000` in a browser.
• For MySQL, replace `YOUR_PASSWORD`, ensure MySQL is running, and create the named database/table before starting
the Node script.
• For MongoDB, ensure `mongod` is running locally. These examples use `mongodb://127.0.0.1:27017/library`.
• For uploads, use a test file. The example upload folders are local and should not be exposed publicly without additional
security checks.
• For POST APIs, use an HTML form or Postman. JSON REST API routes expect JSON body data.
Node.js Practical Question Bank — Solutions Page 9
• Use parameterized SQL (`?` placeholders), validate inputs, and never store plain-text passwords. The login example expects
bcrypt hashes.
• If a port is busy, stop the other server or change `app.listen(3000, ...)` to another available port.
Database setup reference
-- MySQL examples (run in MySQL client)
CREATE DATABASE IF NOT EXISTS college_db;
USE college_db;
CREATE TABLE IF NOT EXISTS students (
 id INT PRIMARY KEY,
 name VARCHAR(100),
 course VARCHAR(50),
 marks INT
);
INSERT INTO students VALUES
(1,'Asha','BCA',88),(2,'Ravi','MCA',81),(3,'Meera','BCA',91);
CREATE DATABASE IF NOT EXISTS company_db;
USE company_db;
CREATE TABLE IF NOT EXISTS Employee (
 eid INT PRIMARY KEY,
 ename VARCHAR(80),
 designation VARCHAR(80),
 company VARCHAR(100),
 salary DECIMAL(10,2)
);
INSERT INTO Employee VALUES (1,'Amit','Developer','ABC Ltd',30000);
Notes on correctness
The question bank does not specify exact database credentials, schema details for every table, or the university's actual
program list. The guide uses clearly marked sample values and standard schemas; adjust them to your lab setup. In the
employee salary example, the Employee table is assumed to contain `eid` and `salary`. In the Student CRUD examples, the
table uses `id, name, course, marks`, matching the first-list specification.
