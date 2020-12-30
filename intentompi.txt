#include <iostream>
#include <cstdlib>
#include <cmath>
#include <ctime>
#include <mpi.h>
#include <string>
#include <vector>
#include <fstream>

using namespace std;
void participantes();

int pos_anio(int anio){
    return(1990-anio);
}




int main(int argc, char** argv ){

    // inicializar variables globales
    int rank;
    int size;

    // Inicializacion de mpi
    MPI_Init(&argc, &argv);

    // Obtencion del rank y el size del cluster
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    // Finalizar en caso de que haya un solo procesador, debido a que este es el orquestador y no tendria maquinas esclavas
    if(size == 1){
        cout << "Solo hay un procesador, se necesitan mas" << endl;
        MPI_Finalize(); 
        return -1;
    }

    // Salario minimo a dolar
    vector<float> smi_dolar;

    // Codigo para obtener los datos del archivo smi.csv
    string linea="";
    string path = argv[1];
    
    std::vector<int> valores_smi, year_smi;
    int valorInt=0;
    int Nproc = 1;
    int cantidad = 0;
    ifstream archivoEntrada(argv[1]);
    MPI_Status status;

    if(rank==0){
        for(linea; getline(archivoEntrada, linea);){

            if(Nproc >= size){
                Nproc = 1;
            }
            cout << linea << endl;

            if(linea[0]!='y'){
                MPI_Send(&linea[0],linea.size()+1,MPI_CHAR,Nproc,0,MPI_COMM_WORLD);
                cantidad += 1;
                Nproc += 1;
            }
            
        }
        Nproc = 1;
        while(cantidad!=0){
            if(Nproc >= size){
                Nproc = 1;
            }
            
            MPI_Recv(&valorInt,1,MPI_INT,Nproc,0,MPI_COMM_WORLD,&status);
            cantidad -= 1;
            valores_smi.push_back(valorInt);
            Nproc +=1;
        } 
    }else{

        
        MPI_Probe(0,0,MPI_COMM_WORLD,&status);
        int count;
        MPI_Get_count(&status,MPI_CHAR,&count);
        char buf [count];
        MPI_Recv(&buf,count,MPI_CHAR,0,0,MPI_COMM_WORLD,&status);
        std::string lineaX = buf; 
        cout << lineaX << endl;
        string value="";
        for(int i=8;i<9999;i++){
            if(lineaX[i]=='"'||lineaX[i]=='.'){

                int valorInt = stoi(value);
                MPI_Send(&valorInt,1,MPI_INT,0,0,MPI_COMM_WORLD);
                break;
            }
            value += lineaX[i];
        }
    }

    // Nproc = 1;
    // cantidad = 0;
    // if(rank==0){
    //     for(linea; getline(archivoEntrada, linea);){

    //         if(Nproc >= size){
    //             Nproc = 1;
    //         }
    //         if(linea[0]!='y'){
    //             MPI_Send(&linea[0],linea.size()+1,MPI_CHAR,Nproc,0,MPI_COMM_WORLD);
    //             cantidad += 1;
    //             Nproc += 1;
    //         }
            
    //     }
    //     Nproc = 1;
    //     while(cantidad!=0){
    //         if(Nproc >= size){
    //             Nproc = 1;
    //         }
            
    //         MPI_Recv(&valorInt,1,MPI_INT,Nproc,0,MPI_COMM_WORLD,&status);
    //         cantidad -= 1;
    //         year_smi.push_back(valorInt);
    //         Nproc +=1;
    //     } 
    // }else{

    //     MPI_Status status;
    //     MPI_Probe(0,0,MPI_COMM_WORLD,&status);
    //     int count;
    //     MPI_Get_count(&status,MPI_CHAR,&count);
    //     char buf [count];
    //     MPI_Recv(&buf,count,MPI_CHAR,0,0,MPI_COMM_WORLD,&status);
    //     std::string lineaX = buf; 

    //     string year="";
    //     for(int i=1;i<9999;i++){
    //         if(lineaX[i]=='"'){
    //             int valorInt = stoi(year);
    //             MPI_Send(&valorInt,1,MPI_INT,0,0,MPI_COMM_WORLD);
    //             break;
    //         }
    //         year += lineaX[i];
    //     }
    // }

    // for(linea; getline(archivoEntrada, linea);){
    //     if(!linea.empty()){
    //         if(linea[0]!='y'){
    //             string value="";

    //             for(int i=8;i<9999;i++){
    //                 if(linea[i]=='"'||linea[i]=='.'){
    //                     valores_smi.push_back(stoi(value));
    //                     break;
    //                 }
    //                 value += linea[i];
    //             }

    //             string year="";
    //             // For para sacar el año de la linea
    //             for(int i=1;i<9999;i++){
    //                 if(linea[i]=='"'){
    //                     year_smi.push_back(stoi(year));
    //                     break;
    //                 }
    //                 year += linea[i];
    //             }
    //         }
    //     }
    // }



    // Codigo para obtener los datos del archivo dollars.csv
    string linea2="";
    string path2 = argv[2];
    ifstream archivoEntrada2(path2);

    // estos vectores contienen los dolares y años en misma posicion como corresponda
    std::vector<float> promedio_dollars, anio_dollars;

    // Se inicializa anio como hola para que al realizar primera comparacion se cumpla y se calcule el promoedio ya que obviamente no existe un año antes del primero
    string anio="hola",anio2="";
    string dolar = "";
    int cont = 0;
    float dolares = 0;
    for(linea2; getline(archivoEntrada2, linea2);){
        if(!linea2.empty()){
            if(linea2[0]!='d'){
                for(int i=1;i<5;i++){
                    anio2 += linea2[i];
                }
                if(anio2==anio||anio[0]=='h'){

                    for(int i=14;i<9999;i++){
                        if(linea2[i]=='"'){
                            dolares += stof(dolar);
                            cont += 1;
                            break;
                        }
                        dolar += linea2[i];
                    } 
                }else{
                    // Realizar division, obtener promedio y añadirlo al vector con los promedios correspondientes
                    promedio_dollars.push_back(dolares/cont);
                    anio_dollars.push_back(stoi(anio));
                    if(stoi(anio)>1994){
                        smi_dolar.push_back(valores_smi[stoi(anio)-1995]/(dolares/cont));
                    }
                    
                    cont = 0;
                    dolares = 0;
                    dolar = "";
                }
                anio="";
                // obtenemos el año de la linea
                for(int i=1;i<5;i++){
                    anio += linea2[i];
                }
            }
            anio2="";

        }
    }

    //Promedio CLP y $US,
    int contador = anio_dollars.size();
    int clp=0,us=0,promedio_clp,promedio_dolares;
    for(int i=0; i< contador ;i++)
    {
        clp += anio_dollars[i];
        us += smi_dolar[i];
    }
    promedio_clp = (clp/contador);
    promedio_dolares = us/contador;
    //beta
    float sxx=0,sxy=0,beta;
    for (int j=0; j < contador ; j++)
    {
        sxy += (anio_dollars[j]-promedio_clp)*(smi_dolar[j]-promedio_dolares);
        sxx += (anio_dollars[j]-promedio_clp)*(anio_dollars[j]-promedio_clp);
    }
    beta = (sxy/sxx);
    //alfa
    float alfa;
    alfa = (promedio_dolares - (beta*promedio_clp));
    //Regresion lineal salario en pesos vs salario en dolares
    //x valor en CLP
    //y valor en $US 
    // For para imprimir el promedio dolar y el año correspondiente
    for(int i = 0;i<year_smi.size();i++){
        cout << "Salario minimo para el año: " << year_smi[i] << " en dolares es: " << smi_dolar[i] << endl;
    }
    cout << endl;
    cout << "=== Resultado ===" << endl;
    cout << endl;
    cout << "y = "<< alfa  << "+ " << beta << "*x" << endl;
    cout << endl;
    cout << "Siendo X el Anio y el valor Y es sueldo promedio en dolares" << endl;
    participantes();
}
void participantes(){
    cout << endl;
    cout << " === Integrantes ===" << endl;
    cout << endl;
    cout << "Javier Gallegos Mancilla" << endl;
    cout << endl;
}

