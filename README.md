#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>
#include <time.h>

#define MAX_PANDAS 100
#define MAX_NOMBRE 80

typedef struct {
    int dia;
    int mes;
    int anyo;
} Fecha;

typedef struct {
    char *nombre;
    float peso;
    int a_nac;
    char sexo;
    Fecha *ingreso;
} Panda;

/* Prototipos */
int edad_Panda(Panda *p);
int ternerBebe(Panda *p);
void imprimePanda(Panda *p);
Panda *buscaPanda(Panda poblacion[], int total, char *nom);
int agregarPanda(Panda poblacion[], int tp);
void listarPandas(Panda poblacion[], int total);
int contarIngresosAnio(Panda poblacion[], int total, int anio);
void listarReproductoras(Panda poblacion[], int total);
void mostrarExtremos(Panda poblacion[], int total);
void liberarPanda(Panda *p);
void liberarPoblacion(Panda poblacion[], int total);
int compararNombres(const char *a, const char *b);

/* Devuelve la edad aproximada usando el año actual. */
int edad_Panda(Panda *p) {
    time_t ahora;
    struct tm *fechaActual;

    if (p == NULL) return -1;

    ahora = time(NULL);
    fechaActual = localtime(&ahora);
    if (fechaActual == NULL) return -1;

    return (fechaActual->tm_year + 1900) - p->a_nac;
}

/* Es candidato si es hembra y tiene más de cinco años. */
int ternerBebe(Panda *p) {
    if (p == NULL) return 0;

    return toupper((unsigned char)p->sexo) == 'H'
        && edad_Panda(p) > 5;
}

/* Imprime todos los datos de un panda. */
void imprimePanda(Panda *p) {
    if (p == NULL) return;

    printf("\nNombre: %s\n", p->nombre);
    printf("Peso: %.2f kg\n", p->peso);
    printf("Anio de nacimiento: %d\n", p->a_nac);
    printf("Sexo: %c\n", p->sexo);

    if (p->ingreso != NULL) {
        printf("Fecha de ingreso: %02d/%02d/%04d\n",
               p->ingreso->dia,
               p->ingreso->mes,
               p->ingreso->anyo);
    }
}

/* Compara nombres sin distinguir mayúsculas y minúsculas. */
int compararNombres(const char *a, const char *b) {
    if (a == NULL || b == NULL) return 0;

    while (*a != '\0' && *b != '\0') {
        if (tolower((unsigned char)*a) !=
            tolower((unsigned char)*b)) {
            return 0;
        }
        a++;
        b++;
    }

    return *a == '\0' && *b == '\0';
}

/* Devuelve la dirección del primer panda cuyo nombre coincide. */
Panda *buscaPanda(Panda poblacion[], int total, char *nom) {
    int i;

    if (poblacion == NULL || nom == NULL) return NULL;

    for (i = 0; i < total; i++) {
        if (compararNombres(poblacion[i].nombre, nom)) {
            return &poblacion[i];
        }
    }

    return NULL;
}

/* Solicita los datos y agrega un panda en la posición tp.
   Devuelve 1 si tuvo éxito y 0 si ocurrió un error. */
int agregarPanda(Panda poblacion[], int tp) {
    Panda nuevo;
    char nombre[MAX_NOMBRE];
    char sexo;
    int leidos;

    if (poblacion == NULL || tp < 0 || tp >= MAX_PANDAS) {
        return 0;
    }

    printf("Nombre: ");
    if (scanf(" %79[^\n]", nombre) != 1) return 0;

    printf("Peso en kg: ");
    if (scanf("%f", &nuevo.peso) != 1 || nuevo.peso <= 0) {
        return 0;
    }

    printf("Anio de nacimiento: ");
    if (scanf("%d", &nuevo.a_nac) != 1) return 0;

    printf("Sexo (H/M): ");
    if (scanf(" %c", &sexo) != 1) return 0;

    sexo = (char)toupper((unsigned char)sexo);
    if (sexo != 'H' && sexo != 'M') {
        printf("Sexo no valido.\n");
        return 0;
    }

    nuevo.nombre = malloc(strlen(nombre) + 1);
    nuevo.ingreso = malloc(sizeof(Fecha));

    if (nuevo.nombre == NULL || nuevo.ingreso == NULL) {
        free(nuevo.nombre);
        free(nuevo.ingreso);
        printf("No hay memoria suficiente.\n");
        return 0;
    }

    strcpy(nuevo.nombre, nombre);
    nuevo.peso = nuevo.peso;
    nuevo.sexo = sexo;

    printf("Fecha de ingreso (dia mes anio): ");
    leidos = scanf("%d %d %d",
                   &nuevo.ingreso->dia,
                   &nuevo.ingreso->mes,
                   &nuevo.ingreso->anyo);

    if (leidos != 3 ||
        nuevo.ingreso->dia < 1 || nuevo.ingreso->dia > 31 ||
        nuevo.ingreso->mes < 1 || nuevo.ingreso->mes > 12 ||
        nuevo.ingreso->anyo < 1) {
        printf("Fecha no valida.\n");
        free(nuevo.nombre);
        free(nuevo.ingreso);
        return 0;
    }

    poblacion[tp] = nuevo;
    printf("Panda registrado correctamente.\n");
    return 1;
}

/* Imprime todos los pandas registrados. */
void listarPandas(Panda poblacion[], int total) {
    int i;

    if (total == 0) {
        printf("No hay registros.\n");
        return;
    }

    for (i = 0; i < total; i++) {
        imprimePanda(&poblacion[i]);
    }
}

/* Cuenta los pandas cuya fecha de ingreso corresponde al año indicado. */
int contarIngresosAnio(Panda poblacion[], int total, int anio) {
    int i;
    int cantidad = 0;

    for (i = 0; i < total; i++) {
        if (poblacion[i].ingreso != NULL &&
            poblacion[i].ingreso->anyo == anio) {
            cantidad++;
        }
    }

    return cantidad;
}

/* Muestra las hembras candidatas al programa de reproducción. */
void listarReproductoras(Panda poblacion[], int total) {
    int i;
    int encontradas = 0;

    for (i = 0; i < total; i++) {
        if (ternerBebe(&poblacion[i])) {
            imprimePanda(&poblacion[i]);
            encontradas = 1;
        }
    }

    if (!encontradas) {
        printf("No hay hembras candidatas.\n");
    }
}

/* Muestra el ejemplar de mayor y menor edad. */
void mostrarExtremos(Panda poblacion[], int total) {
    int i;
    int indiceMayor = 0;
    int indiceMenor = 0;

    if (total <= 0) {
        printf("No hay registros.\n");
        return;
    }

    for (i = 1; i < total; i++) {
        if (edad_Panda(&poblacion[i]) >
            edad_Panda(&poblacion[indiceMayor])) {
            indiceMayor = i;
        }

        if (edad_Panda(&poblacion[i]) <
            edad_Panda(&poblacion[indiceMenor])) {
            indiceMenor = i;
        }
    }

    printf("\nPanda mas viejo:\n");
    imprimePanda(&poblacion[indiceMayor]);

    printf("\nPanda mas joven:\n");
    imprimePanda(&poblacion[indiceMenor]);
}

/* Libera la memoria dinámica de un solo panda. */
void liberarPanda(Panda *p) {
    if (p == NULL) return;

    free(p->nombre);
    free(p->ingreso);

    p->nombre = NULL;
    p->ingreso = NULL;
}

/* Libera la memoria de toda la población. */
void liberarPoblacion(Panda poblacion[], int total) {
    int i;

    for (i = 0; i < total; i++) {
        liberarPanda(&poblacion[i]);
    }
}

/* Programa principal con menú de consultas. */
#ifndef PRUEBAS
int main(void) {
    Panda poblacion[MAX_PANDAS];
    int total = 0;
    int opcion;
    int anio;
    char nombre[MAX_NOMBRE];
    Panda *encontrado;

    do {
        printf("\n===== SISTEMA DE REGISTRO DE PANDAS =====\n");
        printf("1. Agregar panda\n");
        printf("2. Listar todos los pandas\n");
        printf("3. Buscar panda por nombre\n");
        printf("4. Contar ingresos por anio\n");
        printf("5. Ver hembras candidatas a reproduccion\n");
        printf("6. Mostrar panda mas viejo y mas joven\n");
        printf("0. Salir\n");
        printf("Selecciona una opcion: ");

        if (scanf("%d", &opcion) != 1) {
            printf("Entrada no valida.\n");
            break;
        }

        switch (opcion) {
            case 1:
                if (total < MAX_PANDAS) {
                    if (agregarPanda(poblacion, total)) {
                        total++;
                    }
                } else {
                    printf("Se alcanzo el limite de registros.\n");
                }
                break;

            case 2:
                listarPandas(poblacion, total);
                break;

            case 3:
                printf("Nombre del panda: ");
                if (scanf(" %79[^\n]", nombre) == 1) {
                    encontrado = buscaPanda(poblacion, total, nombre);
                    if (encontrado != NULL) {
                        imprimePanda(encontrado);
                    } else {
                        printf("Panda no encontrado.\n");
                    }
                }
                break;

            case 4:
                printf("Anio de ingreso: ");
                if (scanf("%d", &anio) == 1) {
                    printf("Ingresaron %d pandas en %d.\n",
                           contarIngresosAnio(poblacion, total, anio),
                           anio);
                }
                break;

            case 5:
                listarReproductoras(poblacion, total);
                break;

            case 6:
                mostrarExtremos(poblacion, total);
                break;

            case 0:
                printf("Saliendo del sistema...\n");
                break;

            default:
                printf("Opcion no valida.\n");
        }

    } while (opcion != 0);

    liberarPoblacion(poblacion, total);
    return 0;
}
#endif
pruebas.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

/* Se reutilizan las definiciones y prototipos de practica3.c.
   Para compilar este archivo junto con practica3.c, se recomienda
   mover las definiciones de Fecha, Panda y los prototipos a panda.h. */

#include "panda.h"

void prueba_edad_Panda(void) {
    Panda p;
    time_t ahora = time(NULL);
    struct tm *actual = localtime(&ahora);

    p.a_nac = actual->tm_year + 1900 - 8;

    printf("Prueba edad_Panda: ");
    printf("Edad calculada = %d\n", edad_Panda(&p));
}

void prueba_ternerBebe(void) {
    Panda hembra, macho;
    time_t ahora = time(NULL);
    struct tm *actual = localtime(&ahora);

    hembra.a_nac = actual->tm_year + 1900 - 8;
    hembra.sexo = 'H';

    macho.a_nac = actual->tm_year + 1900 - 8;
    macho.sexo = 'M';

    printf("Prueba ternerBebe:\n");
    printf("Hembra de 8 anios: %d (esperado 1)\n",
           ternerBebe(&hembra));
    printf("Macho de 8 anios: %d (esperado 0)\n",
           ternerBebe(&macho));
}

void prueba_imprimePanda(void) {
    Fecha fecha = {10, 5, 2024};
    char nombre[] = "Mei";
    Panda p = {nombre, 95.5f, 2018, 'H', &fecha};

    printf("Prueba imprimePanda:\n");
    imprimePanda(&p);
}

void prueba_buscaPanda(void) {
    Fecha fecha = {10, 5, 2024};
    char nombre1[] = "Mei";
    char nombre2[] = "Tian";
    char buscar[] = "mei";

    Panda poblacion[2] = {
        {nombre1, 95.5f, 2018, 'H', &fecha},
        {nombre2, 110.0f, 2016, 'M', &fecha}
    };

    printf("Prueba buscaPanda:\n");
    printf("Nombre encontrado: %s\n",
           buscaPanda(poblacion, 2, buscar)->nombre);
    printf("Nombre inexistente: %s\n",
           buscaPanda(poblacion, 2, "NoExiste") == NULL
           ? "NULL" : "Encontrado");
}

void prueba_contarIngresosAnio(void) {
    Fecha f1 = {1, 1, 2023};
    Fecha f2 = {2, 2, 2023};
    Fecha f3 = {3, 3, 2024};
    char n1[] = "A";
    char n2[] = "B";
    char n3[] = "C";

    Panda poblacion[3] = {
        {n1, 90, 2018, 'H', &f1},
        {n2, 100, 2017, 'M', &f2},
        {n3, 80, 2020, 'H', &f3}
    };

    printf("Prueba contarIngresosAnio: %d (esperado 2)\n",
           contarIngresosAnio(poblacion, 3, 2023));
}

void prueba_listarPandas(void) {
    Fecha fecha = {10, 5, 2024};
    char nombre[] = "Mei";
    Panda poblacion[1] = {
        {nombre, 95.5f, 2018, 'H', &fecha}
    };

    printf("Prueba listarPandas:\n");
    listarPandas(poblacion, 1);
}

void prueba_listarReproductoras(void) {
    time_t ahora = time(NULL);
    struct tm *actual = localtime(&ahora);
    Fecha fecha = {10, 5, 2024};
    char nombre[] = "Mei";

    Panda poblacion[1] = {
        {nombre, 95.5f, actual->tm_year + 1900 - 8, 'H', &fecha}
    };

    printf("Prueba listarReproductoras:\n");
    listarReproductoras(poblacion, 1);
}

void prueba_mostrarExtremos(void) {
    time_t ahora = time(NULL);
    struct tm *actual = localtime(&ahora);
    Fecha fecha = {10, 5, 2024};
    char nombre1[] = "Mayor";
    char nombre2[] = "Menor";

    Panda poblacion[2] = {
        {nombre1, 100, actual->tm_year + 1900 - 15, 'M', &fecha},
        {nombre2, 80, actual->tm_year + 1900 - 3, 'H', &fecha}
    };

    printf("Prueba mostrarExtremos:\n");
    mostrarExtremos(poblacion, 2);
}

void prueba_liberarPanda(void) {
    Panda p;
    p.nombre = malloc(4);
    p.ingreso = malloc(sizeof(Fecha));

    strcpy(p.nombre, "Pao");
    printf("Prueba liberarPanda:\n");
    liberarPanda(&p);
    printf("Nombre y fecha liberados: %s\n",
           p.nombre == NULL && p.ingreso == NULL ? "SI" : "NO");
}

void prueba_liberarPoblacion(void) {
    Panda poblacion[1];

    poblacion[0].nombre = malloc(4);
    poblacion[0].ingreso = malloc(sizeof(Fecha));
    strcpy(poblacion[0].nombre, "Pao");

    printf("Prueba liberarPoblacion:\n");
    liberarPoblacion(poblacion, 1);
    printf("Memoria liberada correctamente.\n");
}

int main(void) {
    prueba_edad_Panda();
    prueba_ternerBebe();
    prueba_imprimePanda();
    prueba_buscaPanda();
    prueba_contarIngresosAnio();
    prueba_listarPandas();
    prueba_listarReproductoras();
    prueba_mostrarExtremos();
    prueba_liberarPanda();
    prueba_liberarPoblacion();

    return 0;
}
panda.h
#ifndef PANDA_H
#define PANDA_H

typedef struct {
    int dia;
    int mes;
    int anyo;
} Fecha;

typedef struct {
    char *nombre;
    float peso;
    int a_nac;
    char sexo;
    Fecha *ingreso;
} Panda;

int edad_Panda(Panda *p);
int ternerBebe(Panda *p);
void imprimePanda(Panda *p);
Panda *buscaPanda(Panda poblacion[], int total, char *nom);
int agregarPanda(Panda poblacion[], int tp);
void listarPandas(Panda poblacion[], int total);
int contarIngresosAnio(Panda poblacion[], int total, int anio);
void listarReproductoras(Panda poblacion[], int total);
void mostrarExtremos(Panda poblacion[], int total);
void liberarPanda(Panda *p);
void liberarPoblacion(Panda poblacion[], int total);
int compararNombres(const char *a, const char *b);

#endif
compilar-y-ejecutar.sh
gcc -std=c11 -Wall -Wextra -pedantic practica3.c -o practica3
./practica3
compilar-pruebas.sh
gcc -std=c11 -Wall -Wextra -pedantic pruebas.c practica3.c -DPRUEBAS -o pruebas
./pruebas
