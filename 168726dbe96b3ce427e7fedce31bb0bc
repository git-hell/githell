const SYNC_SUB_INTERVAL = 5 * 60000;
const cron = require('node-cron');

const express = require('express');
const fs = require('fs');

const app = express();
app.use(express.static('public'));
app.use(express.urlencoded());

const entries = JSON.parse(fs.readFileSync('entries.json'));
let submissions = JSON.parse(fs.readFileSync('submissions.json'));

app.get('/entries', (req, res) => {
    res.send(JSON.stringify(entries));
});

app.get('/get/submissions', (req, res) => {
    res.send(JSON.stringify(submissions));
});

app.post('/submit', (req, res) => {
    console.log('New entry:', req.body.entry);
    submissions.push(req.body.entry);
    console.log('Syncing submissions');
    fs.writeFileSync('submissions.json', JSON.stringify(submissions));
    res.redirect('/');
});

cron.schedule('30 20 * * *', () => {
    if (submissions.length > 0) {
        console.log('Choosing submission');
        const choice = submissions[Math.floor(Math.random() * submissions.length)];
        entries.push(choice);
        submissions = [];
        fs.writeFileSync('submissions.json', JSON.stringify(submissions));
        console.log('Syncing');
        fs.writeFileSync('entries.json', JSON.stringify(entries));
    } else {
        console.log('No submissions');
    }
});

setInterval(() => {
    console.log('Syncing submissions');
    fs.writeFileSync('submissions.json', JSON.stringify(submissions));
}, SYNC_SUB_INTERVAL);

app.listen(8080, () => {
    console.log('Server up!');
});
