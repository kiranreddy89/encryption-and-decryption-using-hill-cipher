#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>
#include <time.h>

#define MOD 26

int determinant(int key[2][2]) {
    return (key[0][0] * key[1][1] - key[0][1] * key[1][0]);
}

int modInverse(int num, int mod) {
    num = num % mod;
    for (int x = 1; x < mod; x++) {
        if ((num * x) % mod == 1)
            return x;
    }
    return -1;
}

void generateRandomKey(int key[2][2]) {
    int det, detInv;

    do {
        key[0][0] = rand() % MOD;
        key[0][1] = rand() % MOD;
        key[1][0] = rand() % MOD;
        key[1][1] = rand() % MOD;

        det = determinant(key);
        det = (det % MOD + MOD) % MOD;
        detInv = modInverse(det, MOD);
    } while (det == 0 || detInv == -1);

    printf("Generated Key Matrix:\n");
    printf("| %2d %2d |\n| %2d %2d |\n", key[0][0], key[0][1], key[1][0], key[1][1]);
}

void inverseMatrix(int key[2][2], int invKey[2][2]) {
    int det = determinant(key);
    det = (det % MOD + MOD) % MOD;
    int detInv = modInverse(det, MOD);

    invKey[0][0] = ( key[1][1] * detInv) % MOD;
    invKey[1][1] = ( key[0][0] * detInv) % MOD;
    invKey[0][1] = (-key[0][1] * detInv) % MOD;
    invKey[1][0] = (-key[1][0] * detInv) % MOD;

    if (invKey[0][1] < 0) invKey[0][1] += MOD;
    if (invKey[1][0] < 0) invKey[1][0] += MOD;
}

void encrypt(int key[2][2], char *message) {
    int length = strlen(message);
    if (length % 2 != 0) {
        printf("Warning: Message length is odd. The last character will be ignored during encryption.\n");
        length--;
    }

    printf("Encrypted text: ");
    for (int i = 0; i < length; i += 2) {
        int msgVector[2], cipherVector[2];
        for (int j = 0; j < 2; j++)
            msgVector[j] = tolower(message[i + j]) - 'a';

        for (int j = 0; j < 2; j++) {
            cipherVector[j] = 0;
            for (int k = 0; k < 2; k++)
                cipherVector[j] += key[j][k] * msgVector[k];
            cipherVector[j] %= MOD;
        }

        for (int j = 0; j < 2; j++)
            printf("%c", cipherVector[j] + 'a');
    }
    printf("\n");
}

void decrypt(int key[2][2], char *cipher) {
    int length = strlen(cipher);
    int invKey[2][2];
    inverseMatrix(key, invKey);

    printf("Decrypted text: ");
    for (int i = 0; i < length; i += 2) {
        int cipherVector[2], plainVector[2];
        for (int j = 0; j < 2; j++)
            cipherVector[j] = tolower(cipher[i + j]) - 'a';

        for (int j = 0; j < 2; j++) {
            plainVector[j] = 0;
            for (int k = 0; k < 2; k++)
                plainVector[j] += invKey[j][k] * cipherVector[k];
            plainVector[j] = (plainVector[j] % MOD + MOD) % MOD;
        }

        for (int j = 0; j < 2; j++)
            printf("%c", plainVector[j] + 'a');
    }
    printf("\n");
}

int main() {
    int key[2][2];
    char message[100], cipher[100];

    srand(time(NULL));
    generateRandomKey(key);

    printf("Enter a message (uppercase or lowercase): ");
    scanf("%s", message);

    encrypt(key, message);

    printf("Enter the encrypted text: ");
    scanf("%s", cipher);

    decrypt(key, cipher);

    return 0;
}
