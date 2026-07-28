Amazon Web Services — [[Cloud Provider]], який продає compute, storage, networking і купу готових сервісів поверх глобальної мережі [[Data Center|Data Centers]].

Один із найбільших public cloud провайдерів (разом із Azure і GCP). Ресурси береш через консоль / CLI / API, платиш переважно за споживання.

*Деталі інфри й сервісів — окремими нотатками. Нижче — 6 benefits, як їх формулює AWS.*

#### Six benefits of the AWS Cloud

1. **Trade fixed expense for variable expense (pay as you go)**  
   On-prem: великий upfront (приміщення, залізо, люди, upkeep) і фіксований рахунок незалежно від utilization.  
   AWS: старт без CAPEX на свій DC; рахунок місяцями плаває під фактичне споживання. Billing/budgets tools допомагають ловити overrun.

2. **Benefit from massive economies of scale**  
   AWS купує залізо й будує DC у величезних обсягах → нижча собівартість за одиницю → частина економії йде клієнту в ціні.

3. **Stop guessing capacity**  
   On-prem: купуєш під прогноз на роки → або overbuy (гроші в простої), або underbuy (деградація / втрата юзерів, поки ждеш нові сервери тижнями/місяцями).  
   AWS: provision під зараз; scale up/down за хвилини під денний demand.

4. **Increase speed and agility**  
   Підняв test/experiment середовище → перевірив ідею → видалив ресурси і перестав платити. Менше часу на provision/deprovision, більше на innovate.

5. **Stop spending money running and maintaining data centers**  
   Окрім CAPEX на будівництво: racking, stacking, power, охолодження, ремонт — на стороні AWS. Твій фокус — продукт і клієнти, не фізика серверної.

6. **Go global in minutes**  
   On-prem експансія в іншу країну = свої DC / партнери, місяці–роки.  
   AWS: deploy у потрібний Region (наприклад India), інфру там тримає AWS — хвилини замість будівництва з нуля.

#### On-prem vs AWS (стисло)
| | On-prem [[Data Center]] | AWS Cloud |
|---|---|---|
| Витрати | CAPEX + фіксований OPEX | Variable, pay as you go |
| Capacity | Вгадуєш наперед | Scale за demand |
| Експерименти | Дорого / повільно зняти | Швидко підняти й видалити |
| Глобал | Будуй/орендуй DC локально | Deploy у Region |

*Підсумок AWS: cost savings + time savings + доступ до вже збудованої global infrastructure. Те саме логічно стосується моделі [[Cloud Computing]] загалом; формулювання «шістки» — з AWS.*

#### Зв'язки
1. [[Cloud Provider]] — AWS як приклад CSP
2. [[Cloud Computing]] — модель, поверх якої крутяться ці benefits
3. [[Data Center]] — фізика під Regions
4. [[IaaS]] / [[PaaS]] / [[SaaS]] — як упаковані послуги
