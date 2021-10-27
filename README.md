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

Приведём скриншоты отчётов из fastQC и multiQC по мсходным чтениям (также полный отчёт можно увидеть в файле *data/multiqc_report_1.html*, а все скриншоты в папке *images*):  
![](https://github.com/TheMostKnown/hse21_hw1/blob/main/images/Adapter_content_1.png)  
![](https://github.com/TheMostKnown/hse21_hw1/blob/main/images/General_statisctics_1.png)  
![](https://github.com/TheMostKnown/hse21_hw1/blob/main/images/Per_seq_qual_scores_1.png)  
![](https://github.com/TheMostKnown/hse21_hw1/blob/main/images/Seq_qual_hist_1.png)  
![](https://github.com/TheMostKnown/hse21_hw1/blob/main/images/Sequence_Counts_1.png)  

Теперь воспользуемся <code>platanus_trim</code> и <code>platanus_internal_trim</code> для подрезания чтения по качеству:

    platanus_trim sub1.fq sub2.fq
    platanus_internal_trim sub1_mp.fq sub2_mp.fq
    
Теперь удалим исходные данныые, так как они нам больше не понядобятся:

    rm oil_R1.fastq
    rm oil_R2.fastq
    rm oilMP_S4_L001_R1_001.fastq
    rm oilMP_S4_L001_R2_001.fastq
    rm sub1.fq
    rm sub2.fq
    rm sub1_mp.fq
    rm sub2_mp.fq

Теперь с помощью fastQC и multiQC оценим качество подрезанных чтений:

    mkdir trimmed_fastqc
    ls *sub* | xargs -tI{} fastqc -o trimmed_fastqc {}
    mkdir trimmed_multiqc
    multiqc -o trimmed_multiqc trimmed_fastqc

Далее для анализа получившихся файлов, я скачал данные с сервера к себе на компьютер:

    scp -P group_port -r login@server_ip:/home/aakosmachev/hw1/trimmed_fastqc ~/
    scp -P group_port -r login@server_ip:/home/aakosmachev/hw1/trimmed_multiqc ~/

Приведём скриншоты отчётов из fastQC и multiQC по подрезанным чтениям (также полный отчёт можно увидеть в файле *data/multiqc_report_2.html*, а все скриншоты в папке *images*):  
![](https://github.com/TheMostKnown/hse21_hw1/blob/main/images/Adapter_content_2.png)  
![](https://github.com/TheMostKnown/hse21_hw1/blob/main/images/General_statisctics_2.png)  
![](https://github.com/TheMostKnown/hse21_hw1/blob/main/images/Per_seq_qual_scores_2.png)  
![](https://github.com/TheMostKnown/hse21_hw1/blob/main/images/Seq_qual_hist_2.png)  
![](https://github.com/TheMostKnown/hse21_hw1/blob/main/images/Sequence_Counts_2.png)  

Далее соберем континги из подрезанных чтений:

    platanus assemble -f sub1.fq.trimmed sub2.fq.trimmed 2> logfile.log

Собираем скаффолды из контигов, а также из подрезанных чтений:

    platanus scaffold -c out_contig.fa -IP1 sub1.fq.trimmed sub2.fq.trimmed -OP2 sub1_mp.fq.int_trimmed sub2_mp.fq.int_trimmed 2> scaffold.log

Уменьшаем кол-во гэпов с помощью подрезанных чтений:

    platanus gap_close -c out_scaffold.fa -IP1 sub1.fq.trimmed sub2.fq.trimmed -OP2 sub1_mp.fq.int_trimmed sub2_mp.fq.int_trimmed 2> gapclose.log

