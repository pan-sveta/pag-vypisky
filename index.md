# 2. přednáška
## Tasky
- Při návrhu paralelního algoritmu musím prvně zadefinovat **tasky**, které se provádí paralelně
- Tasky se velikostně liší
- **Task dependency graph** - DAG - vyjadřuje závislost prováděných tasků
- **Granuality** - množství tasků, do kterých problém dekompunuju
- **Fine-grained decomposition** - velké množství tasků
- **Coarse grained decomposition** - malé množství tasků

## Pojmy
- **Degree of concurrency** - Počet tasků, které mohou být prováděny paralelně
- **Maximum degree of concurrency** - Maximální počet tasků, které mohou být prováděny paralelně
- **Average degree of concurrency** 
  - Průměrný počet tasků, které mohou být prováděny paralelně po dobu běhu algoritmu
  - Počet tasků / critical path length
- Degree of concurrency je závislé na granuality
- **Critical Path Length** - Nejdelší možná cesta, kterou běh algoritmu vykoná v rámci task dependency graphu
- Granualitu nelze snižovat do konečna - každý problém má *maximální možnou granualitu*
- Tasky spolu musí často komunikovat -> overhead - přílišná granualita nemusí být žádoucí

### Příklad
![Critical path](./img/2_critical_path.png)

- (a)
  - Critical path length - 3 (4 nebo 3 -> 6 -> 7)
  - Shortest parallel execution time - 27
  - Processors count for minimum parallel execution time - 4
  - Maximum degree of concurrency - 4
  - Average degree of concurrency - 7/3 ❓
- (b)
  - Critical path length - 4 (2 nebo 1 -> 5 -> 6 -> 7)
  - Shortest parallel execution time - 28
  - Processors count for minimum parallel execution time - 2
  - Maximum degree of concurrency - 4
  - Average degree of concurrency - 7/4 ❓

## Task Interaction Graphs
- Graf komunikace tasků
- Tasky spolu často musí komunikovat (vyměňovat data), tuto komunikaci lze znázornit grafem

### Příklad
- Představme si násobení sparse matice **A** vektorem **b**
- Potřebujeme násobit pouze nenulové prvky z matice **A** (znázorněno černou tečkou)
- Každý task má přiřazen řádek matice **A** a jeden prvek vektoru **b**
- Např. pro provedení násobení prvního řádku se musíme doptat tasků 1,4,8 na jeho prvek z **b**, protože náš řádek zde obsahuje nenulové číslo a my ho potřebujeme vynásobit s příslušným prvkem z *b*, který náš task ale nezná

![Critical path](./img/2_task_interaction_graph.png)

## Techniky dekompozice
Jak dekomponujeme problém na tasky?

Rozlišujeme čtyři základní techniky
1. [Recursive decomposition](#recursive-decomposition)
2. [Data decomposition](#data-decomposition)
3. [Exploratory decomposition](#exploratory-decomposition)
4. [Speculative decomposition](#speculative-decomposition)

### Recursive decomposition
- Vhodná pro problémy divide-and-conquer (eg. quick sort)
- Rekursivně dekomponujeme problém dokud nedosáhneme požadované granuality

#### Příklad
Quicksort
- Každý sub-list může být rekurzivně zpracováván paralelně
![Recursive decomposition](./img/2_recursive-decomposition.png)

Hledání minima v listu
- Můžeme to pochopitelně dělat sériově
- Paralelně list 4,9,1,7,8,11,2,12 můžeme zpracovat následovně
![Recursive decomposition](./img/2_recursive-decomposition_min.png)

### Data decomposition
- Spoléha na charakteristiku dat, s kterými algoritmus pracuje
- Dekompunujeme tyto data na jednotlivé tasky
- Volba dekompozice značně ovlivňuje výkon algoritmu
- Více přístupů
    1. Output data partitioning
    2. Input data partitioning
    3. Intermediate data partitioning

#### Output Data Decomposition
- Element výsledku je spočten nezávisle na ostatních výsledcích
- Pro stejný problém může existovat více output dekompozic

##### Příklad
Násobení dvou matic
- Každý výsledek **C** je nzávislý na ostatních **C**
![Output Data Decomposition](./img/2_output_data_decomposition_mat.png)

Itemset
- Chceme spočítat počet výskytů itemsetu v databázi transakcí (a)
- To můžeme dekomponovat, tak že rozdělíme itemsety do více tasků (b)
- Nepotřebujeme žádnou komunikaci mezi tasky
![Output Data Decomposition](./img/2_output_data_decomposition_items.png)

#### Input Data Partitioning
- Výstup je spočten jako funkce ze vstupu
- V mnoha případech jediná možná dekomnozice - nevíme co je výstup (např. hledání minima, sort listu)
- Tasku je přiřazena část vstupních data

##### Příklad
Itemset
- Rozdělíme databázi transakcí do více tasků
- Výsledky z jednotlivých tasků poté agregujeme do konečného výsledku

![Input Data Decomposition](./img/2_input_data_decomposition_items.png)

#### Input and Output Data Partitioning
- Kombinace obou přístupů

![Input Data Decomposition](./img/2_inout_data_decomposition_items.png)

#### Intermediate Data Partitioning 
- Sekvence transformací ze vstupních na výstupní data
- Počítáme "mezivýsledky" které následně tranformuje na výsledek
- Často přívější task dependency graph -> rychlejší


![Intermediate Data Partitioning](./img/2_intermediate_data_decomposition_mat.png)

### Exploratory Decomposition
- Dekompozice probíhá společně s průběžnými výpočty
- Použitelmé pro problémy které zahrnují explorativní hledání v prostoru řešení
  - 0/1 integer programming
  - QAP - Quadratic assignment problem
  - Theorem proving
  - Hry (např. šachy)

#### Příklad
15 puzzle
- Posouváme prvky v 2d gridu tak, aby byly seřazené
- Paralelně můžeme v rámci každého tasku počítat všechny možné kroky a najít ty, které jsou správné (na obrázku je správný výsledek podbarven šedě)

![Exploratory Decomposition](./img/2_exploratory_decomposition.png)

#### Anomalous Computations 
- V mnoha případech může explorativní dekompozice měnit množství vykonané práce v širokém rozsahu
- Tzn. že může stejný algoritmus být pro nějaké instance velmi rychlý (sublinear speedup) a pro jiné velmi pomalý (superlinear speedup)❓

![Exploratory Decomposition](./img/2_exploratory_decomposition_anomalous.png)

### Hybrid Decompositions 
- Mix dekompozičních technik
- Často nutný pro dekompozici problémů
- Quicksort 
   - Pouze rekurzivní dekompozice limituje maximální možnou paralelizaci
   - Lepší je použít mix data a rekurzivní dekompozice
   - To je způsobeno tím, že ❓ (hádám, že lepší než pivotovat dlouhý list je lepší to prostě rozesrat na víc procesorů rovnou)
 - Hledání minima
   - Opět je lepší použít mix data a rekurzivní dekompozice

![Hybrid Decomposition](./img/2_hybrid_decomposition.png)

## Tasky
- Problém dekomponujeme na tasky
- Charakteristictika těchto tasků značně ovlivňuje výkon algoritmu
- Faktory:
  - **Task generation (Generování tasků)** - Jak je tvoříme
  - **Task sizes (Velikost tasků)** - Jak jsou tasky velké
  - **Size of data associated with tasks (Množství asociovaných dat)** - Kolik dat jednotlivé tasky zpracovávají

### Task generation (generování tasků)
#### Static task generation
- Víme, kolik tasků bude potřeba na začátku běhu algoritmu
- Např. maticové operace, grafové algortimy, zpracovávání obrázků
- Pravidelně strukturované problémy

#### Dynamic task generation:
- Generujeme tasky v průběhu běhu algoritmu
- Např. 15 puzzle board
- Exploratory nebo speculative decomposition

### Task sizes
#### Uniform
- Jednotná velikost tasků
#### Non-uniform
- Různě velké tasky
- Nevíme, jak budou tasky velké
- Např. discrete optimization problems

### Mapping Techniques
- Tasky musíme přidělovat (mapovat) na fyzické procesory (jádra)
- Toto musí být uděláno chytře
- Minimalizujeme overheads
  - Způsobený komunikací
  - Způsobený nečiností (nepřidělenou prací) některého procesoru
- Minimalizace je optimalizační problém a často si odporuje - např. přidělením celé práce jedinému procesoru minimalizuje komunikace, lale způsobí nečinost všech ostatních procesorů

#### Mapping Techniques for Minimum Idling
 - Musíme současně minimalizovat indling a provádět load balancing
 - Pouhý loadbalancing neminimalizuje idling!
 - Mapování je ovlivěno velikostí tasků, zpsůbem generování, velikostí asociovaných dat
 - Případ (a) je otpimální, případ (b) není

![Mapping Techniques](./img/2_mapping.png)

Dva základní přístupy
1. Static mapping (statické mapování)
   - Předem určíme který task bude zpracovávat jaký procesor
   - Musíme znát (nebo alespoň odhadnout) velikost tasků
   - Stejně se jedná většinou o NP-complete problém
2. Dynamic Mapping
   - Tasky přidělujeme po dobu běhu
   - Použijeme když např. generujeme tasky za běhu nebo neznáme jejich velikost

Komplexita
- Obecně se často jedná o NP-Complete problém
- Tasky mají závislosti
  - Na jeden procesor - P
  - Uniform tasky na 2 a více procesorů - NP-complete
  - Non-uniform tasky na 2 a více procesorů - NP-Complete
- Bez závislostí
  - Na jeden procesor - P
  - Non-uniform tasky na 2 a více procesorů - NP-complete
  - Uniform tasků na 2 a více procesorů - P

### Schemes for Static Mapping 
#### Mappings Based on Data Partitioning
TODO
