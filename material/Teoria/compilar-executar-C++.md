# Compilar/Executar Códigos C++

**Pré-requisitos:**

- Visual Studio Code (VSCode) instalado

### Passos para Windows

1. **Instalar o Compilador** 
2. **Instalar Extensões Necessárias no VSCode:**
    - Abra o VSCode.
    - Vá para a aba de extensões (ícone de quadrado no lado esquerdo).
    - Pesquise e instale a extensão:
        - **C/C++** (Microsoft)

### Passos para Linux

1. **Instalar o Compilador G++:**
    - Abra o terminal e execute o seguinte comando para instalar o G++:
        
        ```
        sudo apt-get update
        sudo apt-get install g++
        
        ```
        
2. **Instalar Extensões Necessárias no VSCode:**
    - Abra o VSCode.
    - Vá para a aba de extensões (ícone de quadrado no lado esquerdo).
    - Pesquise e instale as seguintes extensão:
        - **C/C++** (Microsoft)

### Passos para macOS

1. **Instalar o compilador:**
2. **Instalar Extensões Necessárias no VSCode:**
    - Abra o VSCode.
    - Vá para a aba de extensões (ícone de quadrado no lado esquerdo).
    - Pesquise e instale a extensões:
        - **C/C++** (Microsoft)

### Compilando um Exemplo em C++ para Testar

**Crie um arquivo `main.cpp` com o seguinte conteúdo:**

```cpp
#include <iostream>
using namespace std;

int main() {
    cout << "Hello, World!" << endl;
    return 0;
}

```

### Windows → Compilar e Executar

```
g++ main.cpp -o main.exe
./main.exe

```

### **Linux** → Compilar e Executar:

```
g++ main.cpp -o main
./main

```

### **MacOS** → Compilar e Executar**:**

```
g++ main.cpp -o main
./main

```

Seguindo esses passos, você deve ser capaz de compilar e executar programas C++ em Windows, Linux ou macOS usando o VSCode.