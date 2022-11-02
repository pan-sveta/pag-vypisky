# PAG
## 2. přednáška
### Tasky
- Při návrhu paralelního algoritmu musím prvně zadefinovat **tasky**, které se provádí paralelně
- Tasky se velikostně liší
- Task dependency graph - DAG - vyjadřuje závislost prováděných tasků
- **Granuality** - množství tasků, do kterých problém dekompunuju
- **Fine-grained decomposition** - velké množství tasků
- **Coarse grained decomposition** - malé množství tasků

### Pojmy
- **Degree of concurrency** - Počet tasků, které mohou být prováděny paralelně
- **Maximum degree of concurrency** - Maximální počet tasků, které mohou býtp rováděny paralelně
- **Average degree of concurrency** - Průměrný počet tasků, které mohou býtp rováděny paralelně po dobu běhu algoritmu
- Degree of concurrency je závislé na granuality
- **Critical Path Length** - 
