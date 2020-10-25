# SOLID

## Принцип единственной ответственности (single responsibility principle)

> Класс должен иметь одну и только одну причину для изменения.
>
> Роберт Мартин

Данный принцип означает — класс должен решать определенную проблему и эта проблема должна быть единственной причиной для изменения.

К примеру, в игре у персонажа есть **статы**, и чтобы не потерять набранный опыт и монеты необходимо сохранить информацию в файл.

```c#
struct Stats {
    public string name;
    public int gold;
    public double experience;
}

class Player {
    private Stats stats;

    public Player (Stats stats) {
        this.stats = stats;
    }

    public void SaveStatsToFile (string path) {
        // Save to file...
    }
}
```

В данном примере класс **Player** отвечает за информацию игрока и за ее сохранение. 

Рано или поздно могут измениться правила сохранения, к примеру информацию можно сохранять в JSON формат, можно в XML формат или в любой другой. Из-за этого класс Player будет подвержен изменениям слишком часто, что может привести к ошибкам. 

Чтобы снять с класса Player ответственность за сохранения, лучшим способом будет создать отдельный класс, который будет отвечать за сохранение информации.

```c#
struct Stats {
    public string name;
    public int gold;
    public double experience;
}

class Player {
    private Stats stats;

    public Player (Stats stats) {
        this.stats = stats;
    }

    public Stats GetStats () {
        return stats;
    }
}

class StatsWriter { // Отвечает только за запись
    public void WriteStatsToJSON (Stats stats, string path) {
        // Save to file...
    }

    public void WriteStatsToXML (Stats stats, string path) {
        // Save to file...
    }
}

class StatsReader { // Отвечает только за чтение
    public Stats ReadStatsFromJSON (string path) {
        // Read from file...
        throw new NotImplementedException ();
    }

    public Stats ReadStatsFromXML (string path) {
        // Read from file...
        throw new NotImplementedException ();
    }
}
```

В примере обязанность чтения и записи разделена на два класса - **StatsReader** и **StatsWriter**. Данный классы можно спокойно расширять не трогая класс **Player**.

В данном случае классы решают схожие задачи (только чтение, только запись), одного определенного пользователя (класса Stats). Если со временем изменятся правила сохранения и чтения информации, изменениям будут подвержены только классы чтения/записи.

Принцип единной ответственности не означает, что класс должен содержать только один метод. Главное, что его методы направлены на решения одной или схожей задачи.

Данный принцип может применяться не только к классам. В книге "Чистая Архитектура", Роберт Мартин рассматривает данный прицип на компонентах/сборках (dll, jar, gem). Один компонент должен быть направлен на решения задач связанных с одним пользователем.

> Модуль должен отвечать за одного и только за одного пользователя или заинтересованное лицо.  
>
> Чистая архитектура - Роберт Мартин

Если в программе для введения предприятия есть модуль, который отвечает за бухгалтерию, он должен решать только их проблему, соответственно нужды отдела кадров или склада будут решать другие модули.

## Принцип открытости/закрытости (open–closed principle)

> Сущности (классы, модули, функции...) должны быть открыты для расширения и закрыти для изменения.
>
> Бертран Мейер

Данный принцип ведёт к тому, чтобы код изменялся как можно меньше. Проблема в том, что при постоянном изменении класса необходимо заново тестировать поведение старых функций, чтобы убедиться что новое поведение не привело к новым ошибкам в старом функционале, а потом тестировать новый функционал, чтобы убедиться что он работает без ошибок. Соответственно нужно как-то добавить новый функционал не сломав старый.

К примеру, вы разрабатываете игру и вам необходимо создать оружие. Вы решили создать меч и несколько его видов. В зависимости от вида меча будет зависить его урон.

```c#
enum SwordType {
    Default,
    OrcSword,
}

class Sword {
    private SwordType type;

    public Sword (SwordType type) {
        this.type = type;
    }

    public double GetDamage () {
        switch (type) {
            case SwordType.Default:
                return 10;
            case SwordType.OrcSword:
                return 15;

            default: return 10;
        }
    }
}

class Program {
    public static void Main (string[] args) {
        Sword sword = new Sword (SwordType.Default);
        Console.WriteLine (sword.GetDamage()); // 10
        
        sword = new Sword (SwordType.OrcSword);
        Console.WriteLine (sword.GetDamage()); // 15
    }
}
```

Со временем вы решили добавить новые виды мечей, урон которых будет отличаться, и класс **Sword** снова будет подвержен изменениям:

```c#
enum SwordType {
    Default,
    OrcSword,
    FireSword,
    LongSword,
}

class Sword {
    private SwordType type;

    public Sword (SwordType type) {
        this.type = type;
    }

    public double GetDamage () {
        switch (type) {
            case SwordType.Default:
                return 10;
            case SwordType.OrcSword:
                return 15;
            case SwordType.FireSword:
                return 25;
            case SwordType.LongSword:
                return 20;

            default: return 10;
        }
    }
}


class Program {
    public static void Main (string[] args) {
        Sword sword = new Sword (SwordType.LongSword);
        Console.WriteLine (sword.GetDamage());

        sword = new Sword (SwordType.FireSword);
        Console.WriteLine (sword.GetDamage());
    }
}
```

Чем больше вы будете добавлять видов мечей, тем больше вам придется изменять класс **Sword**, а чем больше изменений, тем больше шансов появления ошибок.

При помощи наследования и полиморфизма может создать базовый класс, наследоваться от него и расширить его поведение:

```c#
class Sword {
    public virtual double GetDamage () {
        return 10;
    }
}

class OrcSword : Sword {
    public override double GetDamage () {
        return 15;
    }
}

class FireSword : Sword {
    public override double GetDamage () {
        return 25;
    }
}

class LongSword : Sword {
    public override double GetDamage () {
        return 20;
    }
}

class MagicSword : Sword {
    private MagicGem gem;

    public MagicSword (MagicGem gem) {
        this.gem = gem;
    }

    public override double GetDamage () {
        return 20 + gem.GetDamage();
    }
}

class MagicGem {
    public double GetDamage () {
        return 10;
    }
}

class Program {
    public static void Main (string[] args) {
        Sword sword = new Sword ();
        Console.WriteLine (sword.GetDamage()); // 10

        sword = new OrcSword ();
        Console.WriteLine (sword.GetDamage()); // 15

        sword = new FireSword ();
        Console.WriteLine (sword.GetDamage()); // 25

        sword = new LongSword ();
        Console.WriteLine (sword.GetDamage()); // 20
        
        sword = new MagicSword (new MagicGem());
        Console.WriteLine (sword.GetDamage()); // 30
    }
}
```

Итак, возможности класса были расширены, но базовый класс был не тронут.

Более реальным примером может служить подписки на разных сайтах, на которых может быть несколько версий с разной ценной, к примеру: Personal, PersonalPro, Team, Enterprise. Каждая следущая подписка содержит в себе возможности предыдущей и добавляет новые. 

В версии **Team** могут быть добавлены инструменты для работы команды, вроде: видеозвонков, общих документов, чатов и так далее. В **Enterprise** версии может быть расширен контроль над безопасностью, экспортом настроек, баз данных или добавлены предварительные версии новых функций.

## Принцип подстановки Барбары Лисков (Liskov substitution principle)

> Пусть q(x) является свойством, верным относительно объектов x некоторого типа T. Тогда q(y) также должно быть верным для объектов y типа S, где S является подтипом типа T.
>
> Барбара Лисков

Роберт Мартин переформулировал определение Барбары Лисков:

> Функции, которые используют базовый тип, должны иметь возможность использовать подтипы базового типа, не зная об этом.
>
> Роберт Мартин

В строготипизированных языках такое поведение реализуется при помощи полифорфизма.

Исходя из предыдущего примера с мечами, добавим класс убийцы — Slayer, который может пользоваться только мечами, и только одним одновременно.

```c#
class Slayer {
    private DefaultSword defaultSword;
    private GoblinSlayerSword goblinSlayerSword;
    
    public void ChangeSword (DefaultSword sword) { 
        defaultSword  =  sword;
    	goblinSlayerSword  =  null;
    }
    
    public void ChangeSword (GoblinSlayerSword sword) { 
        goblinSlayerSword = sword;
    	defaultSword  =  null;
    }
    
    public double GetDamage () {
    	if (goblinSlayerSword != null)
    		return goblinSlayerSword.GetDamage (); 
        else if (defaultSword != null)
   			return defaultSword.GetDamage (); 
        return 0;
    }
}

class DefaultSword {
	private double damage = 15; 
    public double GetDamage () {
		return damage;
	}
}

class GoblinSlayerSword { 
    private double damage = 25; 
    public double GetDamage() {
		return damage;
	}
}

class Program {
    static void Main (string[] args) {
        Slayer slayer = new Slayer();
        slayer.ChangeSword (new DefaultSword());
    	Console.WriteLine (slayer.GetDamage()); // 15

        slayer.ChangeSword (new GoblinSlayerSword());
        Console.WriteLine (slayer.GetDamage()); // 25
    }
}
```

Проблема в том, чтобы пользоваться несколькими мечами, необходимо создавать несколько переменных и проводить дополнительные проверки в методе **GetDamage()**. Чем больше будет классов мечей, тем чаще будет подвергаться изменениям класс **Slayer**.

Благодаря полиморфизму, можно создать базовый класс и наследовать от него все подвиды мечей, а в классе **Slayer** в качестве поля объявить родительский класс **Sword**.

```c#
class Slayer {
	private Sword sword; // используется родительский класс

    public void ChangeSword (Sword sword) { 
    	this.sword = sword;
    }
    
    public double GetDamage () { 
        return sword.GetDamage();
    }
}

abstract class Sword {
	private double damage;
    
    protected void SetDamage (double damage) { 
        this.damage = damage;
    }
    
    public double GetDamage () {
        return damage;
    }
}

class DefaultSword : Sword {
    public DefaultSword () { 
        this.SetDamage (15);
    }
}

class GoblinSlayerSword : Sword { 
    public GoblinSlayerSword () {
		this.SetDamage (25);
	}
}

class FireSword : Sword {
	private double fireDamage = 15; 
    public FireSword () {
		this.SetDamage (15 + fireDamage);
	}
}

class Program {
    static void Main (string[] args) {
        Slayer slayer = new Slayer();
        slayer.ChangeSword (new DefaultSword());
        Console.WriteLine (slayer.GetDamage()); // 15
        
        slayer.ChangeSword (new GoblinSlayerSword());
        Console.WriteLine (slayer.GetDamage()); // 25

        slayer.ChangeSword (new FireSword());
        Console.WriteLine (slayer.GetDamage()); // 30
    }
}
```

Благодаря полиморфизму можно подставлять любой подкласс вместо базового и все будет работать нормально.

И весь смысл **Принципа подстановки Лисков** в том, чтобы вместо родительского класса можно было подставить его дочерний класс.

Пример на структуре данных:

```c#
using System;
using System.Collections.Generic;

class Weapon {}
class Sword : Weapon {}
class Gun : Weapon {}

class MainClass {
    public static void Main (string[] args) {
        List<Weapon> weapons = new List<Weapon> ();
        weapons.Add (new Weapon ());
        weapons.Add (new Sword ());
        weapons.Add (new Gun ());
        Console.WriteLine (weapons.Count); // 3
    }
}
```

## Принцип разделения интерфейса (interface segregation principle)

> **Программные сущности** не должны зависеть от методов, которые они не используют.
>
> Роберт Мартин

Данный принцип очень похож на **Принцип единственной ответственности**, только относится к интерфейсам.



Запомните, что интефрейс должен определять **клиент**. 

## Принцип инверсии зависимостей (dependency inversion principle)

> - Модули верхних уровней не должны зависеть от модулей нижних уровней. Оба типа модулей должны зависеть от абстракций.
> - Абстракции не должны зависеть от деталей. Детали должны зависеть от абстракций.
>
> 





## Дополнительный материал

1. Чистая Архитектура - Роберт Мартин
2. [Простое объяснение принципов SOLID - Хабр](https://habr.com/ru/company/mailru/blog/412699/) 
3. [Принципы SOLID в C# - Proffesorweb](https://professorweb.ru/my/it/blog/net/solid.php)
4. [Принципы SOLID C# - CodeBlog](https://shwanoff.ru/solid-c-sharp/)