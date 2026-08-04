# Архитектура системы голосований 
<img width="1005" height="957" alt="изображение" src="https://github.com/user-attachments/assets/90ddb7c2-d299-4da7-b05a-28a772107715" />


### Создание голосования

Управление сообществом и принятие коллективных решений часто сталкивается с рядом проблем, которые без должной системы будут сильно тормозить развитие или более того убивать сообщество.
Одними из таких проблем выходят: импульсивность, эмоциональность, просто мусорные, заведомо злокачественные голосования которые тратят время участников. 

Данный документ описывает архитектуру систему голосований изначально созданную для бота *RedVote*, хотя она может быть распространена и за его пределы. 

Обозначим, что пользователь имеет право использовать для голосования любой текст от любого человека подходящий под стандарт ACI *(Adequacy Check Instructions)*,
позже я более подробно объясню, что это такое, пока просто держите в уме, что ACI — некоторый стандарт и набор правил для проверки легитимности текста голосований и определения тира, про него тоже поговорим.
Итак, процесс начинается с того, что пользователь формулирует текст голосования. В нем автор обязан исчерпывающе раскрыть тему, причину обсуждения, желаемую цель и предлагаемый вариант решения.
После пользователь выбирает категорию голосования, к примеру: [правосудие], выбранная категория должна напрямую соответствовать обсуждаемой и разрешаемой теме вопроса.
Рассмотрим как это будет выглядеть для пользователя, к примеру, [Discord](https://discord.com/):

Текст: #Текст сообщения 
```
Я предлагаю создать новую роль «Объявления» (без административных прав) и добавить её в меню самостоятельного получения ролей.
В данный момент объявления сообщества отправляются с использованием общего тега,
что создает неудобства и отвлекает пользователей, не заинтересованных в их чтении.
Введение данной роли позволит отправлять целевые уведомления только заинтересованным участникам,
что снизит информационную нагрузку на остальных пользователей и повысит структурное удобство сервера.  
```
ID текста: `1234567890` #ID закреплённый за сообщением

poll text_id: 1234567890 category: роли #Команда инициатор голосования 

— Мы не просто так определяли категорию, в данной архитектуре я поделил голосования на 4 тира, к каждому (уровню) к каждому из них относиться некоторое кол-во категорий.
Категория лишь удобный ориентир пользователя, гораздо проще соотнести текст напрямую с буквенной категорией, а соотношение самой категории с тиром оставить за алгоритмом.
В рамках данной архитектуры тиры могут и их настройки могут различаться от сообщества к сообществу, это, по большей мере, выбор персональный.
Пример тиров:

```
I. Low

Требуется процентов для победы: > 50%
Время на охлаждение: Nope
Кворум по активным пользователям: 15%
Время проведения: 24h

- **`[Медиа]` / `[Косметика]`** — Добавление новых эмодзи, стикеров, смена аватарки или баннера сервера, изменение цветов ролей.
    
- **`[Ивенты]`** — Предложения собраться поиграть, посмотреть фильм, провести временный конкурс на выходных.
    
- **`[Опросы]`** — Голосования без последствий для сервера (например: «Какая часть игры лучше?», «Что смотрим на выходных?»).

II. Mid

Требуется процентов для победы: > 55%
Время на охлаждение: 3h
Кворум по активным пользователям: 30%
Время проведения: 2d

- **`[Каналы]`** — Создание, удаление, переименование текстовых или голосовых каналов. Перемещение каналов по категориям.
    
- **`[Роли]`** — Создание новых *пользовательских* ролей.
    
- **`[Формат]`** — Введение мелких традиций или расписаний.

III. High

Требуется процентов для победы: > 60%
Время на охлаждение: 12h
Кворум по активным пользователям: 50%
Время проведения: 3d

- **`[Правосудие]`** — Голосования за бан, кик, пермач или длительный мут конкретного пользователя. Также сюда относятся амнистии (разбан).
    
- **`[Правила]`** — Добавление новых пунктов в официальные правила сервера, изменение тяжести наказаний за нарушения.
    
- **`[Стафф]`** — Выборы новых модераторов, хелперов, админов. Голосования за лишение модератора его прав (импичмент).
    
- **`[Интеграции]`** — Добавление на сервер ботов, которым требуются широкие права (удаление сообщений, выдача ролей, баны).

IV. Crit

Требуется процентов для победы: > 80%
Время на охлаждение: 24h
Кворум по активным пользователям: 75%
Время проведения: 7d

- **`[Мета]`** — Изменение работы самой системы голосований (изменение процентов кворума, времени КД, стандартов ACI).
    
- **`[Фундамент]`** — Смена основной тематики сервера (например, переход от группы к крупному сообществу).

```

— В данной таблице есть следующие параметры:
```
Требуется процентов для победы
Время на охлаждение
Кворум по активным пользователям
Время проведения
```

— Давайте я объясню, что это более подробно:
**Требуется процентов для победы** — это минимально допустимое отношение суммарного веса голосов "за" к весу голосов "против", необходимое для утверждения инициативы.

Исход голосования определяется процентным соотношением веса положительных голосов к общей массе проголосовавших. 
Например, если суммарный вес "за" составил 52, а вес "против" — 142, общий пул весов равен 194.

В данном сценарии инициатива набрала лишь 26.8% поддержки. Если для текущего типа опроса (тира) правилами установлен минимальный порог одобрения в 50% или 60%, 
система автоматически признает голосование несостоявшимся (не прошедшим порог).

**Время на охлаждение** — обязательная системная пауза, ограничивающая возможность мгновенного голосования. Данная мера внедрена для защиты от импульсивных решений и эффекта толпы. 
Без периода охлаждения сообщество уязвимо перед эмоциональными манипуляциями, когда участники выносят поспешные приговоры (устраивают "линчевание") под влиянием момента, 
не разобравшись в контексте.

**Кворум по активным пользователям** — необходимый минимум пользователей для того, чтобы голосование вошло в силу. Если мы имеем 100 активных пользователей, для, к примеру, IV тира
нужно чтобы проголосовало 75% т.е. 75 человек. Кворум задаёт лишь минимум голосов для вступления в силу предложения, для достижения этого минимума учитываются как активные пользователи, 
так и не очень.
Вот математика подсчёта активных пользователей:
```
Берётся статистика отправленых сообщений за 30 дней, отбрасываются все пользователи с 0 сообщений за 30 дней, на основе тех, кто остался строится масив, 
а его медиана определяет порог активности. Всё то кол-во участников которое выше порога считается активным кол-во пользователей
```
— RedVote, к примеру, логирует дату, время и пользователя который отправил сообщение без сохранения контента. На основе этих данных за каждым пользователем в настоящее время закрепляется
определённое кол-во сообщений, эти количества представляют из себя масив по которому и находится медиана, все сообщения отправленные 30 днями ранее удаляются.

**Время проведения** — время на протяжении которого длиться голосование, чем выше тир, тем выше важность, тем больше даётся времени подумать и задействовать участников.

Вот мы и определили в каком формате создаётся голосование, как у него определяется параметры по тирам и каким методом они рассчитываются. Поговорим про саму систему голосования.
После создания голосования оно более не принадлежит одному человеку, оно является общественным, системным делом под авторством пользователя, при чём автор текста и пользователь инициации
может быть совершенно разным. В голосовании есть 5 вариантов:
```
Полностью согласен (+2)
Согласен (+1)
Воздержусь/Нейтрально (+0)
Не согласен (-1)
Полностью не согласен (-2)
```

Значения в скобках: "(+-X)" — это вес голоса, он считается отдельно от кол-во самих голосов, к примеру для достижения кворума. Я предпочитаю использовать 5-вариативную весовую систему 
из-за её гибкости и возможности измерять, хоть и посредственно, настроение участников.

---

### Система проверки ACI

Вся выше описанная система определённо не так плоха, но в ней чего-то не хватает, как не пропускать чрезмерно эмоциональные, не логичные и просто токсичные голосования в маcсы?
— Нужен максимально объективный способ проверки который был бы доступен каждому пользователю. 
В данной архитектуре им стал:
```
Adequacy Check Instructions (ACI) — стандарт и набор правил для проверки легитимности текста голосований и определения тира. 
*Примечание:* Термин "Instructions" здесь используется в широком смысле. В сллегитимности текста голосоваучае, если проверку осуществляет ИИ, 
ACI выступает не как жесткий текстовый алгоритм, а как набор граничных значений (порогов), метрик оценки (например, допустимый уровень токсичности или спама) и системных ограничений, 
в рамках которых модель принимает решение о валидности текста.
```
То есть под ACI понимается набор данных который каким-либо способом может быть использован для проверки текста на структурную (логическую) адекватность.

К примеру вот вариант текстового ACI:
```
ADEQUACY CHECK INSTRUCTIONS (ACI) – CORE PROTOCOL

EVALUATION MANDATE AND OUTPUT REQUIREMENT:
Your evaluation must culminate in exactly two strictly formatted outputs based on the logical, structural, and categorical analysis of the submitted text.
* Legitimacy [0 or 1]: Return 1 (TRUE/VALID) if and only if the text unconditionally satisfies ALL criteria outlined in Sections I through V. Return 0 (FALSE/INVALID) if the text violates even a single sub-clause of this protocol.
* Theme of the text [1, 2, 3, or 4]: You must definitively classify the structural intent of the text into one of four systemic Tiers (defined in Section V). 

***

SECTION I: LINGUISTIC AND SEMANTIC COHERENCE (LEXICAL AXIOMS)
To be validated, the text must utilize language exclusively for the transfer of coherent information.
1. Syntactic Integrity: The combination of words must follow standard grammatical and syntactic rules. Word salads, randomized character strings, or intentionally obfuscated phrasing immediately nullify the proposal.
2. Absurdity & Post-Irony Filter: The text must not rely on internet post-irony, meme-based logic, or surrealism. The proposal must exist within the objective reality of the community's operations.
3. Semantic Clarity: What is explicitly written is all that exists. The proposal cannot rely on "unwritten context," inside jokes, or assumed knowledge. The core intent must be immediately comprehensible to a standard user.

SECTION II: LOGICAL STRUCTURE AND CAUSALITY (THE XYZ TRIAD)
A legitimate proposal must demonstrate an unbroken logical chain. The text must satisfy the following causal formula: X + Y -> Z (Action X, applied to Status Quo Y, rationally produces Expected Outcome Z).
1. Actionability (X): The text must clearly define a concrete, executable action. It cannot be an abstract wish, a rhetorical question, or a vague complaint.
2. Causal Justification (Y): The text must state an objective operational reason for the action. Proposals submitted "just for fun," "because I want to," or "to see what happens" fail the causality check (unless explicitly categorized under Tier 1 Cosmetic/Fun).
3. Logical Consequence (Z): The stated expected outcome must rationally follow from the proposed action. If there is a logical disconnect between the action and the goal (e.g., "Remove the moderation team so the server's ping improves"), the proposal is invalid.

SECTION III: INFORMATIONAL NEUTRALITY AND OBJECTIVE FRAMING
The text of a vote must serve strictly as a neutral ballot, not a propaganda leaflet or an emotional vector.
1. Absence of Emotional Coercion: The text must not contain emotional blackmail, guilt-tripping, manipulative phrasing, or passive-aggressive ultimatums (e.g., "Vote yes if you actually care about this community").
2. Prohibition of Loaded Terminology: The framing must be completely stripped of subjective, inflammatory, or inherently biased adjectives designed to pre-determine the voter's stance. 
   - Invalid: "Do you agree to permanently ban the toxic, power-hungry user John?"
   - Valid: "Do you agree to issue a permanent ban to user John for repeated rule violations?"
3. Ad Hominem Ban: The proposal must address structural issues, documented actions, or community protocols. It must never attack the personal character, inherent traits, or external/real life of community members.

SECTION IV: SYSTEMIC PRESERVATION
The democratic mechanism shall not be utilized to execute the destruction of the platform that hosts it.
1. Systemic Suicide: Any vote aimed at the deletion of the community, mass-banning of the user base without specific operational cause, or deliberate sabotage of the digital infrastructure is instantly classified as a zero-state (0) proposition.

SECTION V: CATEGORY ALIGNMENT AND TIER VERIFICATION
The text must be analyzed to determine its true systemic impact. You must assign the correct Tier based on the text's actual logical objective. If the text attempts to combine multiple actions across different Tiers, it must be classified by the highest applicable Tier.
* TIER 1 (Low Impact / Routine): Temporary events, purely cosmetic changes (emojis, server icons), and casual polls with no systemic consequences. The action does not grant or remove permissions, does not alter server architecture, and does not penalize any user.
* TIER 2 (Medium Impact / Organizational): Spatial and structural convenience. Creation, deletion, or renaming of text/voice channels; addition of standard cosmetic roles; minor disciplinary actions (e.g., short-term mutes not exceeding 2 hours); establishment of minor scheduling traditions.
* TIER 3 (High Impact / Justice and Governance): Restriction of freedoms, distribution of authority, and rule modification. Banning, kicking, or long-term muting of a user; granting or revoking administrative/moderator privileges; integrating complex bots with moderation permissions; amending or adding official community rules.
* TIER 4 (Critical Impact / Foundational Meta): Existential mechanics and core constitution. Altering the voting mechanisms (ACI parameters, cooldowns, quorum percentages); complete server theme shifts; database/economy wipes. 

***

FINAL OUTPUT FORMAT:
Upon processing the text, you must return ONLY the following data structure:
Legitimacy: [0 or 1]
Theme of the text: [Tier 1, Tier 2, Tier 3, or Tier 4]
```

— Он проверяет легитимность текста и соответствие тиров. В архитектуре RedVote каждый участник имеет право отправить запрос на проверку соответствия ACI командой `report <ID text> ACI`.
В целом, можно реализовать проверку NLP до выхода самого голосования в массы, но в данной версии архитектуры я выбрал именно ручную проверку, а реализовывать её для каждого голосования даже
без жалоб, скажем — не рационально. Любой управленец/модератор видя репорт на голосовании может проверить его хоть через тот же ИИ, опять же, право проверки сохраняется за каждым участником,
просто именно модератор имеет право отклонить `reject <ID text> ACI` или одобрить `confirm <ID text> ACI` голосование.
Если голосование не проходит ACI т.е. вывод `Legitimacy 0` и/или `Theme of the text: X` не совпадает с действующим тиром (т.е. уровнем категории) оно отменяется.
Если голосование проходит, т.е. вывод `Legitimacy 1` и `Theme of the text: X` совпадает с действующим тиром оно получает защиту от последующих жалоб. Если же голосование было принято или отклонено по
ошибке, никому из пользователей не составит труда это заметить путём самоличной проверки

Пример на основе текста:
```
Я предлагаю создать новую роль «Объявления» (без административных прав) и добавить её в меню самостоятельного получения ролей.
В данный момент объявления сообщества отправляются с использованием общего тега,
что создает неудобства и отвлекает пользователей, не заинтересованных в их чтении.
Введение данной роли позволит отправлять целевые уведомления только заинтересованным участникам,
что снизит информационную нагрузку на остальных пользователей и повысит структурное удобство сервера.
```
<img width="814" height="650" alt="изображение" src="https://github.com/user-attachments/assets/df731cdb-6711-47f7-bd11-19832b7f0b9b" />

Можно ли отклонить голосование после победы если оно в итоге не соответсвует ACI? — Дело персональное, я склоняюсь к весрии, что отклонить можно после победы, но до реализации. Если что-то уже реализованно
или реализуеться отклонять не вариант, как минимум голосование в любом случае прошло все остальные этапы.

---

Copyright (c) 2026 Kirill S. (headwatch).

Permission is granted to copy, distribute and/or modify this document under the terms of the GNU Free Documentation License, Version 1.3 or any later version published by the Free Software Foundation; with no Invariant Sections, no Front-Cover Texts, and no Back-Cover Texts. A copy of the license is included in the section entitled "GNU Free Documentation License".
