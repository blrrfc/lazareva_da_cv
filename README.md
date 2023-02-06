# Портфолио
## Лазарева Диана Александровна
# Образование
- Сибирский Государственный Университет Телекоммуникаций и Информатики (СибГУТИ), 2 курс, специальность - "ПО мобильный систем"
- Средняя школа
# Навыки
- C
# Выполненные работы
- Курсовая работа "Шифр Плейфера на c"
```C
#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include<malloc.h>

void toLowerCase(char plain[], int ps)//функция для преобразования в нижний регистр
{
	int i;
	for (i = 0; i < ps; i++) {
		if (plain[i] > 64 && plain[i] < 91)//если код символа в ASCII > 64 и < 91 (строчные буквы 97 - 122, заглавные 65 - 90)
			plain[i] += 32; //код символа + 32 (разница между заглавным и соответствующим строчным символом)
	}
}


int removeSpaces(char* plain, int ps)//функция для удаления пробелов
{
	int i, count = 0;
	for (i = 0; i < ps; i++)
		if (plain[i] != ' ')
			plain[count++] = plain[i];//если символ не является пробелом то он записывается
	plain[count] = '\0';//в конец массива добавляется символ конца строки
	return count;
}


void generateKeyTable(char key[], int ks, char keyT[5][5])//формирование матрицы 5х5 для шифрования (5х5 из-за кол-ва букв в англ алфавите)
{
	int i, j, k, flag = 0, * dicty;
	dicty = (int*)calloc(26, sizeof(int));//выделение памяти (26 эл) для хранения алфавита
	for (i = 0; i < ks; i++) {
		if (key[i] != 'j')
			dicty[key[i] - 97] = 2;
	}
	//одна буква обычно заменяется, чаще всего 'j' на 'i'
	dicty['j' - 97] = 1;

	i = 0;
	j = 0;

	for (k = 0; k < ks; k++) {
		if (dicty[key[k] - 97] == 2) {
			dicty[key[k] - 97] -= 1;
			keyT[i][j] = key[k];
			j++;
			if (j == 5) {
				i++;
				j = 0;
			}
		}
	}

	for (k = 0; k < 26; k++) {
		if (dicty[k] == 0) {
			keyT[i][j] = (char)(k + 97);
			j++;
			if (j == 5) {
				i++;
				j = 0;
			}
		}
	}//формирование таблицы 5х5. первые символы составляет ключ, затем остальные символы алфавита
}

void search(char keyT[5][5], char a, char b, int arr[])//поиск символов в таблице(биограммы)
{
	int i, j;

	if (a == 'j')//если 1 из двух букв в биограмме это , то заменяется на
		a = 'i';
	else if (b == 'j')//если второй
		b = 'i';

	for (i = 0; i < 5; i++) {

		for (j = 0; j < 5; j++) {

			if (keyT[i][j] == a) {
				arr[0] = i;//когда находит 1 букву в таблице записывает номера ее строки и столбца в массив
				arr[1] = j;
			}
			else if (keyT[i][j] == b) {
				arr[2] = i;
				arr[3] = j;// когда находит 2 букву
			}
		}
	}
}


int mod5(int a) { return (a % 5); }//мод(:)5


int prepare(char str[], int ptrs)//функция для увеличения длины строки до четного числа
{
	if (ptrs % 2 != 0) {
		str[ptrs++] = 'z';//если нечетное то в конец строки добавляется
		str[ptrs] = '\0';
	}
	return ptrs;
}


void encrypt(char str[], char keyT[5][5], int ps)//шифрование(действия)
{
	int i, a[4];
	for (i = 0; i < ps; i += 2) {
		search(keyT, str[i], str[i + 1], a);
		if (a[0] == a[2]) {//если обе буквы находятся в 1 строке
			str[i] = keyT[a[0]][mod5(a[1] + 1)];
			str[i + 1] = keyT[a[0]][mod5(a[3] + 1)];//свдигаемся по таблице на один элемент вперед(вправо)(дл обеих букв)
		}
		else if (a[1] == a[3]) {//если в 1 столбце
			str[i] = keyT[mod5(a[0] + 1)][a[1]];
			str[i + 1] = keyT[mod5(a[2] + 1)][a[1]];//сдвигаемся по тиблице на один элемент вперед(вниз)
		}
		else {
			str[i] = keyT[a[0]][a[3]];//если нет общей строки/столбца, то номер строки сохраняется, но номер столбца меняется на номер стобца 2 буквы
			str[i + 1] = keyT[a[2]][a[1]]; // (т.е.двигаемся по квадрату)
		}
	}
}


int encryptByPlayfairCipher(char str[], char key[], FILE* f)//шифрование
{
	char ps, ks, keyT[5][5];
	ks = strlen(key);
	ks = removeSpaces(key, ks);
	toLowerCase(key, ks);//преобразование ключа
	ps = strlen(str);
	toLowerCase(str, ps);
	ps = removeSpaces(str, ps);//преобразование текста

	if ((f = fopen("text.txt", "w")) == NULL)
	{
		printf("Error occured while opening file");
		return 1;
	}
	fputs(str, f);
	fclose(f);//открытие файла и запись в него исходного текста
	ps = prepare(str, ps);//преобразование текста

	generateKeyTable(key, ks, keyT);
	encrypt(str, keyT, ps);
}

void decrypt(char str[], char keyT[5][5], int ps)
{
	int i, a[4];
	for (i = 0; i < ps; i += 2) {
		search(keyT, str[i], str[i + 1], a);
		if (a[0] == a[2]) {
			str[i] = keyT[a[0]][mod5(a[1] - 1)];
			str[i + 1] = keyT[a[0]][mod5(a[3] - 1)];
		}
		else if (a[1] == a[3]) {
			str[i] = keyT[mod5(a[0] - 1)][a[1]];
			str[i + 1] = keyT[mod5(a[2] - 1)][a[1]];
		}
		else {
			str[i] = keyT[a[0]][a[3]];
			str[i + 1] = keyT[a[2]][a[1]];
		}
	}
}

// Function to call decrypt
int decryptByPlayfairCipher(char str[], char key[], FILE* f2)
{
	char ps, ks, keyT[5][5];

	// Key
	ks = strlen(key);
	ks = removeSpaces(key, ks);
	toLowerCase(key, ks);

	// ciphertext
	ps = strlen(str);
	toLowerCase(str, ps);
	ps = removeSpaces(str, ps);


	generateKeyTable(key, ks, keyT);

	decrypt(str, keyT, ps);
	//FILE* filed;

	if ((f2 = fopen("decrypt.txt", "w")) == NULL)
	{
		printf("Error occured while opening file");
		return 1;
	}
	fputs(str, f2);
	fclose(f2);
}
int compare(FILE* f, FILE* f2) {
	char s[50];
	char ss[50];
	if ((f = fopen("text.txt", "r")) == NULL)//"r" — открыть файл для чтения
	{
		printf("Error occured while opening file");
		return 1;
	}
	if ((f2 = fopen("decrypt.txt", "r")) == NULL)
	{
		printf("Error occured while opening file");//проверка на открытие файла
		return 1;
	}
	/* while (!feof(f)) {
	fgets(s,sizeof(s),f);
	}
	while (!feof(f2)) {
	fgets(ss,sizeof(ss),f2);
	}
	if (strcmp (s, ss)==0){
	printf("Введенный и дешифрованный тексты совпадают.");
	}
	else {
	printf("Во время шифрования произошла ошибка");
	}
	fclose(f);
	fclose(f2);
	}*/
	while (!feof(f) && !feof(f2))
	{
		//fgets(s,sizeof(s), f);
		//fgets(ss, sizeof(ss), f2);
		fscanf(f, "%s", s);
		fscanf(f2, "%s", ss);
		if (strcmp(s, ss) == 0) {
			printf("Введенный и дешифрованный тексты совпадают.");
		}
		else {
			printf("Во время шифрования произошла ошибка");
		}
		fclose(f);
		fclose(f2);
	}
}



int main()
{

	char* str, * key;
	str = (char*)malloc(sizeof(char) * 50);
	printf("Введите текст для шифрования: ");
	char* whr;
	whr = gets(str);
	FILE* file;
	FILE* filed;
	if ((file = fopen("text.txt", "w")) == NULL)
	{
		printf("Error occured while opening file");
		return 1;
	}//fputs(str, file);
	fclose(file); if ((filed = fopen("decrypt.txt", "w")) == NULL)
	{
		printf("Error occured while opening file");
		return 1;
	}fclose(filed);
	printf("Введите ключ шифрования: ");
	key = (char*)malloc(sizeof(char) * 50);
	scanf("%s", key);
	int e = encryptByPlayfairCipher(str, key, file);

	printf("Cipher text: %s\n", str);
	int d = decryptByPlayfairCipher(str, key, filed);

	// fputs(str, filed);

	printf("Deciphered text: %s\n", str);
	return 0;
	int c = compare(file, filed);
	free(str);
	free(key);
}
```