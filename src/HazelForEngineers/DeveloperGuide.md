<div class="title"> 
    <img src="../res/HazelGradientLogo-Square.png" alt="Hazel Logo" />
    <h1> Developer Guide </h1>
</div>

This document is for Hazel engineers working on the engine.

## Etiquette
The following rules apply to all engineers working on the engine codebase. The motivation behind these is to support a positive collaborative environment, assist in producing a higher quality product for developers and end-users, and keep the code itself as high quality as possible.

### 1. Code Ownership
Being a Game Engine, Hazel naturally divides itself into many various discrete systems which also need to work together well. Typically, these various systems will have _owners_ - engineers who are responsible for a particular system or section of code, who are usually (but not always) the original authors. There may also be some overlap or multiple owners for certain systems. Usually ownership is implicit upon an engineer writing code; that is to say that an engineer automatically becomes the owner of  code that they write (especially true for systems they've designed and implemented). There can always be exceptions to this of course, which falls onto the responsibility of the project's technical directors.

As the name implies, owners are responsible for code which they own. Whenever engineers need to stray into code territory that they _do not own_, and determine that this foreign code needs to be _modified_, it's important that certain processes are followed. These processes can be summarized by just a few points:

- Do _not_ modify any code that you do not own unless there is an important reason to (eg. you need to extend a system to accept new input from a feature you're working on). Needlessly changing code is a problem because whenever _anything_ changes, there could be undesirable side-effects which you couldn't possibly even envision - because you didn't write the code!
- If you've determined that you do in fact have a good reason to change code, **contact the owner and state your intentions**. This is important because the owner responsible for the code will know more about it (and all of its implications, "load-bearing" properties, etc.) than you, and this collaboration can ensure that what you're doing is correct and minimizes any possible negative effects from your changes. It's also just polite to give the owner a heads up, because after all it is _their_ code that you're coming in and changing.
- If there is an emergency - there is a critical bug in the engine that you can fix ASAP even though you don't own the code, contact your Technical Director and inform them of this change.

Just to be clear - you can change code that you own without needing to notify anyone, and ultimately for whatever reason. The burden of responsibility falls on you in that case, because after all you are the owner of the code you're modifying.

### 2. Self-Documenting Code
One of Hazel's major engineering goals is to promote easy-to-read, good, clean code. This is a very subjective description, however the idea is that your code should be as readable as possible, even to "junior" engineers. Of course there are areas that simply cannot be simplified, or by doing so would affect the engine negatively, but in all areas this idea of clear, simple, and self-documenting code should be followed. This is more thoroughly explained in the Code Style section with specific examples, and includes things like naming symbols descriptively, preferring verbose descriptive code to auto-generated templates or functional style programming which produces more abstract engineer-facing code that can be harder to understand and see the full extent of computation taking place.

Your code should ideally be able to read like an English book as much as possible. If you can make your function make more sense by expanding variable names or adding extra comments - do that!

## Code Style
By now Hazel shouldn't really need a Code Style guide, because there is enough existing code in the codebase to answer every stylistic question you might have. Long story short - if you don't know what style to write your code in, look around! There's plenty of code, and your goal is to blend in. Nevertheless, I will try and officially summarize the main points here.

### Naming

Classes, functions, member functions, enums, namespaces, source code file names, and most other "titles" are always in Pascal Case (like the Win32 API and Microsoft's C# code style).

Local variables (including parameters) are always camel case. Private member variables are prefixed with `m_` and then start with a capital letter (eg. `m_VariableName`). Static variables (either in a translation unit or class) are prefixed with an `s_` and also start with a capital letter (eg. `s_StaticVariable`).

Here is an example which demonstrates all of these points:

```cpp

namespace MyNamespace {

    static int s_StaticInt = 0;

    static void MyFunction();

    class ExampleClass
    {
    public:
        void ThisIsMyFunction(int inputVariable)
        {
            m_MemberVariable = inputVariable;

            std::string localVariable = std::to_string(inputVariable);
        }
    private:
        int m_MemberVariable = 0;
    };

}

```

To be continued...