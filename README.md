let fs = require('fs');

function getParametersOfProgram() {
let programMode;
let pathToInputFile;
let pathToOutputFile;

process.argv.forEach((val, index) => {
if (index == 2)
programMode = val;
if (index == 3)
pathToInputFile = val;
if (index == 4)
pathToOutputFile = val;
});

if (programMode != 'code' && programMode != 'decode') {
console.error('ACHTUNG!\nНеверно указан режим работы программы!\nACHTUNG!');
process.exit(-1);
}
if (pathToInputFile == undefined || pathToOutputFile == undefined) {
console.error('ACHTUNG!\nНужно указать пути к считываемому файлу и файлу для вывода результата программы!\nACHTUNG!');
process.exit(-1);
}

return new Array(programMode, pathToInputFile, pathToOutputFile);
}

function code(text) {
let codedText = '';
let escapeSymbol = '#';
let i = 0, n = 1;
let nJump;

while (i < text.length) {
while (text.charAt(i) == text.charAt(i + n))
n++;
nJump = n;

while (n >= 255) {
codedText += escapeSymbol + String.fromCharCode(255) + text.charAt(i);
n -= 255;
}

if (n > 3 || (n > 0 && text.charAt(i) == escapeSymbol))
codedText += escapeSymbol + String.fromCharCode(n) + text.charAt(i);
else
for (j = 0; j < n; j++)
codedText += text.charAt(i);

i += nJump;
n = 1;
}

return codedText;
}

function decode(codedText) {
let decodedText = '';
let escapeSymbol = '#';
let i = 0, n = 1;

while (i < codedText.length) {
if (codedText.charAt(i) == escapeSymbol) {
n = codedText.charCodeAt(i + 1);
for (j = 0; j < n; j++)
decodedText += codedText.charAt(i + 2);
i += 3;
continue;
}
decodedText += codedText.charAt(i);
i++;
}

return decodedText;
}

function perform(option, pathToInputFile, pathToOutputFile) {
fs.readFile(pathToInputFile, (err, data) => {
if (err) {
if (err.code == 'ENOENT') {
console.error('ОШИБКА!\nПути ' + pathToInputFile + ' не существует');
process.exit(-1);
}
if (err.code == 'EISDIR') {
console.error('ACHTUNG!\nОжидался путь до файла\nНо указан путь до каталога!');
process.exit(-1);
}
if (err.code == 'EACCES') {
console.error('ACHTUNG!\nОтказано в доступе к файлу ' + pathToInputFile);
process.exit(-1);
}
}

function calculateCompressionRatio(originalText, codedText) {
console.log('Коэффициент сжатия = ', originalText.length / codedText.length);
}

let text = data.toString();
let newText;
if (option == 'code') {
newText = code(text);
calculateCompressionRatio(text, newText);
}
else {
newText = decode(text);
}

fs.writeFile(pathToOutputFile, newText, (err) => {
if (err) {
if (err.code == 'EACCES')
console.error('ACHTUNG!\nОтказано в доступе к файлу ' + pathToOutputFile);
process.exit(-1);
}
});
});
}

function main() {
let parametersOfProgram = getParametersOfProgram();
let programMode = parametersOfProgram[0];
let pathToInputFile = parametersOfProgram[1];
let pathToOutputFile = parametersOfProgram[2];

if (programMode == 'code')
perform('code', pathToInputFile, pathToOutputFile);
else
perform('decode', pathToInputFile, pathToOutputFile);
}


main();
