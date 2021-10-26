# Домашняя работа 1
По майнору биоинформатика, 2 года обучения.  
Выполнил Космачев Алексей, МИЭМ 3 курс.

После первичных приготовлений к выполнению домашней работы (получение доступа к серверу, создание репозитория и т.д.), подключаемся к серверу по нужному порту и оказываемся в своей папке. Далее вводим следующие комманды для создания папки с домашним заданием и символических ссылок на файлы с самими заданиями:  

    mkdir hw1
    cd hw1
    ln -s /usr/share/data-minor-bioinf/assembly/oil_R1.fastq
    ln -s /usr/share/data-minor-bioinf/assembly/oil_R2.fastq
    ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq
    ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq

Moжно проверить наличие необходимых файлов командой <code> ls -l </code>. Далее переходим непосредственно к выполнению. Для начала возьмём random seed из моего месяца и дня рождения (508). И запустим с этими данными команду <code> seqtk </code> для случайного выбора чтений:

    seqtk sample -s508 oil_R1.fastq 5000000 > sub1.fq
    seqtk sample -s508 oil_R2.fastq 5000000 > sub2.fq
    seqtk sample -s508 oilMP_S4_L001_R1_001.fastq 1500000 > sub1_mp.fq
    seqtk sample -s508 oilMP_S4_L001_R2_001.fastq 1500000 > sub2_mp.fq

Далее, с помощью fastQC и multiQC оценим качество исходных чтений и получим по ним общую статистику:

    mkdir fastqc
    ls *sub* | xargs -tI{} fastqc -o fastqc {}
    mkdir multiqc
    multiqc -o multiqc fastqc

Далее для анализа получившихся файлов, я скачал данные с сервера к себе на компьютер:

    scp -P group_port -r login@server_ip:/home/aakosmachev/hw1/fastqc ~/
    scp -P group_port -r login@server_ip:/home/aakosmachev/hw1/multiqc ~/
