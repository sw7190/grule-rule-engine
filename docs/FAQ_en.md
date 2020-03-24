# FAQ

[Tutorial](Tutorial_en.md) | [Rule Engine](RuleEngine_en.md) | [GRL](GRL_en.md) | [RETE Algorithm](RETE_en.md) | [Functions](Function_en.md) | [Grule Events](GruleEvent_en.md) | [FAQ](FAQ_en.md)

---

## 1. Grule Panicked on Maximum Cycle

**Question** : I got the following panic message when Grule engine is executed.

```
panic: GruleEngine successfully selected rule candidate for execution after 5000 cycles, this could possibly caused by rule entry(s) that keep added into execution pool but when executed it does not change any data in context. Please evaluate your rule entries "When" and "Then" scope. You can adjust the maximum cycle using GruleEngine.MaxCycle variable.
```

**Answer** : The rule engine is done evaluating rule entries for choosing which one to execute in the 5000th time (5000 it the maximum execution cycle). You can change specify any positive number but if you doubt, you an leave it to 5000. When the rule set were evaluated that many times more to this number (the nax cycle) it will panic. To fix this issue, have to understand how rule engine works. The following simulation should help you understand the problem and know how to mitigate it.

Consider this fact.

```go
type Fact struct {
   Payment int
   Cashback int
}
```

The following rules defined.

```text
rule GiveCashback "Give cashback if payment is above 100" {
    When 
         F.Payment > 100
    Then
         F.Cashback = 10;
}

rule LogCashback "Emit log if cashback is given" {
    When 
         F.Cashback > 5
    Then
         Log("Cashback given :" + F.Cashback);
}
```

Then you execute the rule with Fact instance of

```go
&Fact {
     Payment: 500,
}
```

Now when the engine executes....

Cycle 1 : Execute "GiveCashback" .... because when F.Payment > 100 is a valid condition<br>
Cycle 2 : Execute "GiveCashback" .... because when F.Payment > 100 is a valid condition<br>
Cycle 3 : Execute "GiveCashback" .... because when F.Payment > 100 is a valid condition<br>
Cycle 4 : Execute "GiveCashback" .... because when F.Payment > 100 is a valid condition<br>
Cycle n : Execute "GiveCashback" .... because when F.Payment > 100 is a valid condition<br>
Cycle 5000 : Execute "GiveCashback" .... because when F.Payment > 100 is still a valid condition<br>
panic

You should notice Grule execute the same rule again and again because the **WHEN** condition keep yielding a valid result.

One way for this solution is to change "GiveCashback" rule to something like :

```text
rule GiveCashback "Give cashback if payment is above 100" {
    When 
         F.Payment > 100 &&
         F.Cashback == 0
    Then
         F.Cashback = 10;
}
```

This way, after the 1st execution, the rule's WHEN is become invalid and not get executed again.
Or ...

```text
rule GiveCashback "Give cashback if payment is above 100" {
    When 
         F.Payment > 100
    Then
         F.Cashback = 10;
         Retract("GiveCashback");
}
```

"Retract" function will pull the "GiveCashback" rule from knowledge base so it will not be evaluated again
in the next cycle. When you execute the engine again, all retracted rules will be reset.

---

## 2. Saving Rule Entry to database

**Question** : Will there be a feature that enable Grule to load/save rules from database ?

**Answer** : No. While it is a good idea to store your rule entries into a database, Grule will not create any database adapter to automaticaly store and retrieve rule from database.
You can easily create such adapter your self. Grule have provided a common way to load rules into Knowledgebase from *Reader*, *File*, *Byte Array*, *String* and *Git*. Strings can be easily inserted and selected from database, as you load them into Grule's knowledgebase. 

There are so many database can potentially store Rule Entries. Creating adapter for those databases means we need to get committed to their driver updates, every single one of them. So we decide that, if you want to store rule in database, you can create such mechanism your self.

---

## 3. Maximum number of rule in one knowledge-base

**Question** : How many rule entry can be inserted into knowledgebase ?

**Answer** : You can have as many rule entries you want. But there should be at least one minimum.

---