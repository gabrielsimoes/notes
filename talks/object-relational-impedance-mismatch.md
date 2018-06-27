# Object-relational impedance mismatch

This is smart name we came up to call the difficulties that arise when we try to
couple together a RDBMS (namely SQL), and a Object-Oriented Language. We'll
first examine why this problem exists, and then take an attempt at solving it.

## Mismatches

- When exposing a object in a object-relational mapping, we violate some properties of objects, namely [^1]:
  - **Encapsulation**: object properties which are expected in OO to be unexposed are necessarily exposed.
  - **Accessbility**: the relational has no notion of private/public access -- everything is public.
  - **Interface, class, inheritance and polymorphism** are unknown concepts in the relational model, as well as **views**[^2] are something unknown to object orientation, which cannot be properly emulated with interfaces and methods. Still, the relational model has no notion of **behaviour**.
- While OO **structure** is contructed based on links, and the relational model is made of global, unnested relation variables. Also, OO **integrity** is maintained through exceptions, while the relational model uses declarative contraints on the data.
- OO doesn't have a small and well-defined set of **maniplative operators** like the relational model.
- **Transactions** in the relational model can be very large and diverse, whereas in OO they are tipically made of individual assignments.

[^1]: [Wikipedia article on the Object-relational impedance mismatch](https://en.wikipedia.org/wiki/Object-relational_impedance_mismatch).

[^2]: [Wikipedia article on SQL Views](https://en.wikipedia.org/wiki/View_(SQL)).


## Solving the problem according to myself

I'll be honest, my opinion is that there is no way to actually solve this problem. In my view the two solutions are:

- **Using objects**: Ditch relational databases. Use key-value stores,
  document databases or other NoSQL databases. Don't use a relational database
  where it doesn't fit quite well.
- **Using SQL**: Use relational models as they were designed. Relational
  models are designed to *answer questions about your data*, not to simply
  store objects. In this case, you're better of using a Functional Programming
  language.

  We still might have some problems due to mutable data. One way were this
  problem wouldn't exist is when we have an append-only architecture, but check below for other ideas.

Stuart Halloway below describes a nice approach he uses in [Datomic](https://www.datomic.com/).
In my view, his approach fits FP, but not OOP, even though he says otherwise.

## Solving according to Stuart Halloway (one of Datomic creators)
- A nice table from a Stuart Halloway's talk [^3]:

  |                   | OO                      | RDBMS         | We Want    |
  |-------------------|-------------------------|---------------|------------|
  | **Processing**    | object at time          | set at a time | both       |
  | **Structure**     | dictionaries            | rectangles    | any        |
  | **Access**        | navigation (pointers)   | query         | both       |
  | **Location**      | over here               | over there    | anywhere   |
  | **Programmable?** | no                      | no            | yes        |
  | **Perception**    | coordinated             | coordinated   | **VALUES** |
  | **Action**        | assist use of tx system | transaction   | any        |

  [^3]: [The Impedance Mismatch is Our Fault - Stuart Halloway](https://www.infoq.com/presentations/Impedance-Mismatch).

- **The "over there" problem**: many objects have to fetched at the same time
  because data could be changes out of the blue.
- **The "coordinated perception" problem**: we have to use locks and stop
  everything to read data.

- We gotta simplify the two models in order to use them together.
- We can't have everything from the OO and relational models.
- **VALUES** are key to solving this problem. Data is made of values, not
  places. Data is immutable.
- The [Datomic](https://www.datomic.com/) way:
  |                   | Solution      |
  |-------------------|---------------|
  | **Processing**    | any           |
  | **Structure**     | any           |
  | **Access**        | any           |
  | **Location**      | anywhere      |
  | **Programmable?** | yes           |
  | **Perception**    | values        |
  | **Action**        | txes + tx fns |

- **Bonuses**:
  - Time model (data from x time ago).
  - Scale read and query horizontally.
  - "What if" queries.
  - Multi-source queries
