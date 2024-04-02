## Installation

* Repository klonen und submodules installieren: `git clone https://github.com/stefan-strableg/bulme-swnw-cheat-project.git --recurse-submodules`

**ODER** (falls die Git Version auf den Schul-PCs zu alt ist)

* Repository klonen: `git clone https://github.com/stefan-strableg/bulme-swnw-cheat-project.git`
* In das Repository wechseln: `cd bulme-swnw-cheat-project`
* Subomdules initialisieren: `git submodule update --init --recursive`


## Begriffe

**IPC** (Inter-process communication): Kommunikation zwischen Prozessen

**TSL** (Test and Set Lock): Eine Prozessoranweisung, die gleichzeitig (atomar) einen bestimmten Wert liest und dann setzt. Darauf basieren alle Synchronisationsmethoden.

**Race conditions**/ data races: Ein "Wettlauf" von zwei Threads im Zugriff auf dieselben Daten. (Im concurrent programming geht es darum, diese zu vermeiden.)

**Atomare** Operationen: nicht teilbare Operationen (thread-safe, keine race conditions)

**Synchronisation**: Man spricht von Thread-Synchronisation, wenn Strukturen wie Mutex, Semaphor, etc. verwendet werden, in einem Thread zu warten, bis ein anderer Thread ein Signal sendet.

**kritischer Bereich**: Ein Codeabschnitt, in dem der ausführende Thread nicht unterbrochen werden darf.


## **Synchronisationsmethoden** in C#

* **Mutex**

Ein Mutex (**Mut**ual **ex**clusion/ Gegenseitiger Ausschluss) wird verwendet, um sicherzugehen, dass immer nur ein Thread gleichzeitig auf ein Objekt zugreift. Es gibt die folgenden Methoden:

* `WaitOne` legt den Thread schlafen, bis der Thread, der den Mutex gerade sperrt, ihn wieder freigibt. Danach sperrt der aktuelle Thread den Mutex für sich.
* `ReleaseMutex` gibt den Mutex wieder frei, damit ihn andere Threads benutzen können.

Codebeispiel:

```cs
Mutex mutex;

mutex.WaitOne();
criticalFunction();
mutex.ReleaseMutex();
```

---

* **Semaphore**

Ein Semaphore ist wie ein Mutex, nur das er öfter gesperrt werden kann. Er wird verwendet, wenn nicht nur ein Thread gleichzeitig auf eine Ressource zugreifen können soll, sondern mehrere.
Er wird nur selten verwendet und ist deswegen nicht näher beschrieben.

---


* **Monitor**

Monitor hat vier Methoden, die alle auf jedes `object` in C# aufgerufen werden können: `Enter` und `Exit`, `Wait` und `Pulse`.

* `Enter` sperrt Zugriff auf das Objekt wie ein Mutex. Wenn der Zugriff bereits gesperrt ist, schläft der Thread so lange, bis er wieder freigegeben wird.
* `Exit` gibt den Mutex wieder frei
* `lock` wird als kürzere Form für beide verwendet. (Siehe Codebeispiel unten)

---

* `Wait` legt den Thread schlafen, bis ein anderer Thread `Pulse` aufruft
* `Pulse` kann nur von einem Thread aufgerufen werden, wenn er das Objekt vorher gesperrt hat (mit `Enter` oder `lock`)

Codebeispiele:

```cs
Monitor.Enter(obj);
    criticalFunction();
Monitor.Exit(obj);

// oder, in schönerem Syntax:

lock(obj)
{
    criticalFunction();
}
```

```cs
    void thread1loop()
    {
        Monitor.Wait(obj);
        ...
    }

    void thread2loop()
    {
        ...
        lock(obj)
        {
            Monitor.Pulse(obj); 
            ...
        }
        // Thread wacht 1 auf, wenn er gerade im Wait ist
        ...
    }
```
---

## Testfragen

### **Welche Synchronisationsmethoden gibt es?**

Es gibt Lösungen mit **aktivem** und **passivem Warten**. Lösungen mit aktivem Warten sind generell schlecht, da der Thread nicht schläft, sondern aktiv nichts macht und Prozessorzeit verschwendet. Bei passivem Warten schläft der Thread und wird erst aufgeweckt, wenn er ein Signal von einem anderen Thread bekommt.

#### Aktives Warten (Spinlocks)

* **Interrupts ausschalten** (nur für µCs, auf Betriebssystemen schlecht, sprengt alles)
* **Strikter Wechsel** (Warten bis sich eine variable ändert)

#### Passives Warten

* **TSL-Assembleranweisung** (Mutex, Monitor, Semaphor, etc.)

### Race Condition in Assembler

|Thread 1|Thread 2| Index|
|---|---|---|
|LOD AX, Idx||3|
|ADD AX, 1||3|
||LOD AX, Idx|3|
|STO Idx, AX||4|
||ADD AX, 1|4|
||STO Idx, AX|4|

