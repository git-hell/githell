const ROW2_Y = 15;
const PADDING = 2;
const WHITESPACE_WIDTH = 20;

const input = document.getElementById('text');
const canvas = document.getElementById('c');
canvas.width = window.innerWidth;
canvas.height = ROW2_Y + 35;
const c = canvas.getContext('2d');
const { width, height } = canvas;

const cons = [ 'b', 'c', 'ĉ', 'd', 'f', 'g', 'ĝ', 'h', 'ĥ', 'j', 'k', 'l', 'm', 'n', 'p', 'r', 's', 'ŝ', 't', 'ŭ', 'v', 'z'];
const xsubs = {
    'c': 'ĉ',
    'g': 'ĝ',
    's': 'ŝ',
    'h': 'ĥ',
    'u': 'ŭ',
};
const vowels = [ 'a', 'e', 'i', 'o', 'u' ];
const punc = {
    ',': 'comma',
    '.': 'period',
    '?': 'question',
    '!': 'exclamation'
};
const whitespace = [ ' ', '\n', '\t', '\r', '"', '\'' ];

const drawImage = (filename, x, y = null, parent_width = null) => {
    const image = new Image();
    image.src = filename;
    image.onload = () => {
        const diff = parent_width != null ? (parent_width - image.width) / 2 : 0;
        if (y != null) {
            c.drawImage(image, x + diff, y);
        } else {
            c.drawImage(image, x + diff, height / 2 - image.height / 2);
        }
    };
    return image.width;
};

const renderText = (text, vowel = null, x = 0) => {
    if (text.length == 0) {
        if (vowel) {
            console.log(vowel);
            drawImage('carry.png', x, ROW2_Y);
            drawImage(`${vowel}.png`, x, 0);
        }
    } else {
        if (text[1] == 'j' || (text[0] == 'a' && text[1] == 'ŭ')) {
            console.log(text.slice(0, 2));
            if (vowel) {
                console.log(vowel);
                drawImage('carry.png', x, ROW2_Y);
            }
            const add = vowel ? drawImage(`${vowel}.png`, x, 0) + PADDING : 0;
            const width = drawImage(`${text.slice(0, 2)}.png`, x + add);
            renderText(text.slice(2), null, x + add + width + PADDING);
        } else if (text[1] == 'x' && Object.keys(xsubs).includes(text[0])) {
            if (vowel) {
                console.log(vowel);
                drawImage('carry.png', x, ROW2_Y);
            }
            const add = vowel ? drawImage(`${vowel}.png`, x, 0) + PADDING : 0;
            console.log(xsubs[text[0]]);
            const width = drawImage(`${xsubs[text[0]]}o.png`, x + add);
            renderText(text.slice(2), null, x + add + width + PADDING);
        } else if (vowels.includes(text[0])) {
            if (vowel) {
                console.log(vowel);
                drawImage('carry.png', x, ROW2_Y);
            }
            const out = vowel ? drawImage(`${vowel}.png`, x, 0) + PADDING : 0;
            renderText(text.slice(1), text[0], x + out);
        } else if (Object.keys(punc).includes(text[0])) {
            if (text.length == 1) {
                if (vowel) {
                    drawImage('carry.png', x, ROW2_Y);
                }
            }
            const add = ((text.length == 1) && vowel) ? drawImage(`${vowel}.png`, x, 0) + PADDING : 0;
            const width = drawImage(`${punc[text[0]]}.png`, x + add, height - 20);
            renderText(text.slice(1), (text.length != 1) ? vowel : null, x + width + add + PADDING);
        } else if (whitespace.includes(text[0])) {
            renderText(text.slice(1), vowel, x + WHITESPACE_WIDTH);
        } else if (cons.includes(text[0])) {
            console.log(vowel ? vowel + text[0] : text[0]);
            const width = drawImage(`${text[0]}o.png`, x, ROW2_Y - (text[0] == 'v' ? 15 : 0));
            if (vowel) {
                drawImage(`${vowel}.png`, x, 0, width);
            }
            renderText(text.slice(1), null, x + width + PADDING);
        } else {
            renderText(text.slice(1), vowel, x);
        }
    }
};

input.onkeyup = () => {
    c.clearRect(0, 0, width, height);

    const parts = [];
    const text = input.value;

    renderText(text);
};
