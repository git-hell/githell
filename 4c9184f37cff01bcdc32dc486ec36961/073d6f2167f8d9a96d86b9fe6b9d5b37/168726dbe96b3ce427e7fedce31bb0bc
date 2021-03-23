fetch('/get/submissions', {
    method: 'GET'
}).then(r => r.json()).then(data => {
    const ul = document.getElementById('subs');
    console.log(data);
    data.forEach(item => {
        const li = document.createElement('li');
        li.textContent = item;
        ul.prepend(li);
    });
});
