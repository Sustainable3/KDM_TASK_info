# TASK - KDM - Tryton
Marek Drwal, 22.11.2025

## polecenia
`ssh trytonp`

a poza tym podstawy bash

Komunikacja student-STOS: program PuTTY i komenda scp do przesyłania plików (IP: user@trytonp-dmv.task.gda.pl)

### jednorazowo, na początku

    mkdir -p ${TASK_USER_WORK}/.apptainer
    rm -fr /users/kdm/${USER}/.apptainer
    ln --symbolic ${TASK_USER_WORK}/.apptainer /users/kdm/${USER}/.apptainer


### kolejkowanie
Zadania obliczeniowe należy zdefiniować, potem zlecić uruchomienie, na końcu odebrać wynik

## uruchamianie obliczeń

### polecenia do *SLURM*
- `sbatch job-gpu.sh` - dodanie zadania

- `squeue` - stan

- `squeue -u $USER` - zadania użytkownika

- `scancel zad` - usunięcie z kolejki

- `sinfo` - o kolejkach

### przykład

#### kod programu
*gpu_check.py*

    import torch

    def sprawdz_gpu():
        print(f"Wersja PyTorch: {torch.__version__}")
        
        # 1. Sprawdzanie NVIDIA CUDA (Windows/Linux)
        if torch.cuda.is_available():
            print("\n GPU (CUDA) jest dostępne!")
            print(f"Liczba urządzeń: {torch.cuda.device_count()}")
            print(f"Nazwa obecnego GPU: {torch.cuda.get_device_name(0)}")
            device = torch.device("cuda")
            
            
        # 3. Brak GPU
        else:
            print("\n GPU nie jest dostępne. Obliczenia będą wykonywane na CPU.")
            device = torch.device("cpu")
        
        print(f"Aktywne urządzenie: {device}")

    if __name__ == "__main__":
        sprawdz_gpu()
	

#### zadanie w kolejce test
*job-non-gpu-test.sh*

    #!/bin/bash
    #SBATCH -J twoje_imie_test_gpu_1
    #SBATCH -p test
    #SBATCH -N 1
    #SBATCH --mail-type=ALL
    #SBATCH --mail-user=s191684@student.pg.edu.pl

    module load trytonp/apptainer/1.3.0
    singularity exec --nv docker://nvcr.io/nvidia/pytorch:21.11-py3 python3 gpu_check.py

#### zadanie w kolejce z gpu
*job-gpu.sh*

    #!/bin/bash
    #SBATCH -J twoje_imie_test_gpu_2
    #SBATCH --partition=gpu-a100
    #SBATCH --mail-type=ALL
    #SBATCH --mail-user=s191684@student.pg.edu.pl

    module load trytonp/apptainer/1.3.0
    singularity exec --nv docker://nvcr.io/nvidia/pytorch:21.11-py3 python3 gpu_check.py


### Ultralytics przykład

#### definicja zadania
inny Docker image

*job-ultralytics.sh*

    #!/bin/bash
    #SBATCH -J md_eval_1
    #SBATCH --partition=gpu-a100-80gb
    #SBATCH --nodes=1
    #SBATCH --mail-type=ALL
    #SBATCH --mail-user=s191684@student.pg.edu.pl

    module load trytonp/apptainer/1.3.0
    singularity exec --nv docker://ultralytics/ultralytics:latest python3 combined_PV_eval.py

### varia
- `$TASK_APL/example/slurm/`

- `$CITASK_USER_SCRATCH`

- `/users/project1/pt01299/` - katalog naszego projektu

## źródła
- https://docs.task.gda.pl/kdm/aplikacje-narzedziowe-i-systemowe/slurm
- https://docs.task.gda.pl/kdm/poradnik-uzytkownika/pytorch-na-serwerach-gpu/slurm
- https://docs.task.gda.pl/kdm/poradnik-uzytkownika/pytorch-na-serwerach-gpu/pytorch
- https://docs.task.gda.pl/kdm/poradnik-uzytkownika/jak-rozpoczac-obliczenia


- https://docs.task.gda.pl/kdm/poradnik-uzytkownika/linux/edytory
- https://docs.task.gda.pl/kdm/poradnik-uzytkownika/modules
- https://docs.task.gda.pl/kdm/poradnik-uzytkownika/pytorch-na-serwerach-gpu/apptainersingularity
- https://docs.task.gda.pl/kdm/poradnik-uzytkownika/przestrzen-dyskowa
- https://docs.task.gda.pl/kdm/poradnik-uzytkownika/dostep-do-serwerow#Kopiowanie

## oświadczenia

Treść *gpu_check.py* adaptowana z odpowiedzi narzędzia typu GenAI

> Obliczenia wykonano z wykorzystaniem komputerów Centrum Informatycznego Trójmiejskiej Akademickiej Sieci Komputerowej" (Computations were carried out using the computers of Centre of Informatics Tricity Academic Supercomputer & Network).
