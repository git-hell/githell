fetch('/entries', {
    method: 'GET'
}).then(r => r.json()).then(data => {
    const ul = document.getElementById('entries');
    console.log(data);
    data.forEach(item => {
        const li = document.createElement('li');
        li.textContent = item;
        ul.prepend(li);
    });
});
