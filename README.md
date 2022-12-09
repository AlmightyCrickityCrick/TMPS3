# Behavioral Design Patterns


## Professor: Mihai Gaidau
## Student: Scripca Lina

----

## Objectives:

* Study and understand the Behavioral Design Patterns;
* Implement at least 5 BDPs for a specific domain;


## Used Design Patterns:

* Visitor
* Strategy
* State
* Observer
* Memento


## Implementation
The design patterns mentioned earlier have been implemented within the context of two applications that can be found below:

[Domain Specific Language for Genetics Problems Solution](https://github.com/Damyy17/dsl_gen/tree/main/src/main/java/antlr)
[Lost and Found app](https://github.com/AlmightyCrickityCrick/Lost-Found/tree/main/app/src/main/java/com/example/lostfound)

### Visitor

Visitor is a design pattern that lets us separate the algorithms from the objects on which they operate. The visitor within the genetics DSL operates on the DSL Program definitions, visiting the Parsing tree of each program, branch by branch, and exhibiting appropriate dehavior for each one of them:

```
public class Visitor<T> extends GeneticsGrammarBaseVisitor<T>{
    Map<String, IDataType> variables = new HashMap<>();
    InputInfo inputInfo;
    static final String[] keywords = {"genes", "parents", "generation", "set", "dom", "phenotype", "codominance", "location",
    "label", "genotype", "frequency", "square", "find", "cross", "pred", "estimate", "if", "then", "from", "family","gen", "for",
    "else", "end", "while", "do", "print"};
    static final String[] types = {"genes", "parents", "generation", "number", "boolean", "string", "family"};
    static boolean parentAssignedFlag = false;

    @Override
    public T visitChildren(RuleNode node) {
        return super.visitChildren(node);
    }

    public T visitProgram(GeneticsGrammarParser.ProgramContext ctx){
        Map<String, IDataType> variables = new HashMap<>();
        parentAssignedFlag = false;
        inputInfo = new InputInfo();
        visitChildren(ctx);
        return (T) inputInfo.getAllOutput();
    }

    @Override public T visitStatements(GeneticsGrammarParser.StatementsContext ctx) {
        return visitChildren(ctx);}

    @Override
    public T visitDeclaration(GeneticsGrammarParser.DeclarationContext ctx) throws ReservedKeywordException, NonexistentTypeException {
        IDataType value = null;
        List<String> child = new ArrayList<String>();

        for (ParseTree e : ctx.children) {
            if (!Objects.equals(e.getText(), ",") && !Objects.equals(e.getText(), ";"))
                child.add(e.getText());
        }

        if (!Arrays.asList(types).contains(child.get(0)))
            throw new NonexistentTypeException("Nonexistent Type Exception is occurred! " + ctx.getText() + ". " + child.get(0));
        else {
            for (String s : child.subList(1, child.size())) {
                if (Arrays.asList(types).contains(child.get(0))) {
                    switch (child.get(0)) {
                        case "genes":
                            value = new Gene();
                            break;
                        case "parents":
                            value = new Parent();
                            break;
                        case "generation":
                            value = new Generation();
                            break;
                        case "family":
                            value = new Family();
                            break;
                        case "number":
                            value = new DSLNumber();
                            break;
                        case "boolean":
                            value = new DSLBoolean();
                            break;
                        case "string":
                            value = new DSLString();
                            break;
                    }
                    if (!Arrays.asList(keywords).contains(s))
                        variables.put(s.toLowerCase(), value);
                    else
                        throw new ReservedKeywordException("Reserved Keyword Exception is occurred! " + ctx.getText() + ". " + s);
                }
            }
        }

//        throw child);
//        throw variables);
        return super.visitDeclaration(ctx);
    }
//etc etc
```
with the tree itself being created separately after the parsing process of the text introduced in the applications's UI: 

```
                codeWrite.setText(codeWrite.getText());
                CharStream codeExample = CharStreams.fromString(String.valueOf(codeWrite.getText()));
                GeneticsGrammarLexer lexer = new GeneticsGrammarLexer(codeExample);
                GeneticsGrammarParser parser = new GeneticsGrammarParser(new CommonTokenStream(lexer));

                ParseTree tree = parser.program();
                GeneticsGrammarBaseVisitor<String> visitor = new Visitor<>();
                String allOutputInfo = visitor.visit(tree);
```
### Strategy Pattern

Strategy is a behavioral design pattern that lets us define a family of algorithms, put each of them into a separate class and make their objects interchangeable. Within the DSL, a strategy pattern has been implemented for evaluating expressions within Condition Branches.

```
import antlr.DSLExceptions.IncompatibleTypeException;

public interface EvaluationStrategy {
     String eval(Object a, Object b, String sign) throws IncompatibleTypeException;
}

public class EvaluationBoolStrategy implements EvaluationStrategy{

    @Override
    public String eval(Object a, Object b, String sign) throws IncompatibleTypeException {
        return eval((boolean)a, (boolean) b, sign);
    }

    public String eval(boolean value1, boolean value2, String sign) throws IncompatibleTypeException {
        switch (sign){
            case "and":
                if(value1 && value2) return "true";
                else return "false";
            case "or":
                if(value1 || value2) return "true";
                else return "false";
            case "==":
                if(value1 == value2) return "true";
                else return "false";
            case "!=":
                if(value1 != value2) return "true";
                else return "false";
            default:
                throw new IncompatibleTypeException("Incompatible Type Exception is occured! " + value1 + " and " + value2 + " can't be compared through " + sign);
        }
    }
}

```
with the right type of strategy being selected based on variable type

```
case "boolean":
      strateg = new EvaluationBoolStrategy();
// ....
a.setValue("value", strateg.eval(firstb.getValue(), secondb.getValue(), children.get(1)));


```

### State Pattern

State is a behavioral design pattern that lets an object alter its behavior when its internal state changes. This pattern has been implemented within the Lost and Found App for handling requests and updates within the Dashboard, which differ whether the state is set to Lost or Found.

Below is the Dashboard implementation:

```
class Dashboard : Fragment() {
    lateinit var recyclerView:RecyclerView
    lateinit var adapter: AnnouncementRecyclerViewAdapter
    var annArray= DebugConstants.getAnnouncements()
    lateinit var toolbar:androidx.appcompat.widget.Toolbar
    lateinit var state :DashboardState
    //...
    
    private fun modifyState(){
        if(this.state.type == 0) {
            this.state = FoundState(this)
        }
        else {
            this.state = LostState(this)
        }
    }

```
with each of the states containing the following structure:
```
class FoundState(dashboard: Dashboard) : DashboardState(dashboard) {
    override var type = 1
    init {
        initiateStateChange()
    }
    override fun initiateStateChange() {
        setToolbar()
        getAnnouncements()
    }

    override fun setToolbar() {
        var name : TextView = dashboard.toolbar.findViewById(R.id.toolbar_fragment_name)
        name.setText(R.string.found)
    }

    override fun getAnnouncements(){
        runBlocking {
            var job = async { ApolloClientService.getAllFoundAnn() }
            var result = job.await()
            if(result.size>0) {
                dashboard.annArray = result
                dashboard.updateDashboard()
            }

        }
    }
}
```

### Observer Pattern

Observer is a behavioral pattern that lets us define a subscription mechanism to notify multiple bjects about nay events that happen to the object they're observing.

Within the Lost and Found app, an Observer implemented within hte androidx.lifecycle library has been attached within Login and Register Activities to two components of the corresponding ViewModel classes, formState and resultState, where they monitor their states and force the Activity to perform certain actions once the forms achieve a specific state


```
registerViewModel.loginResult.observe(this@RegisterActivity, Observer {
            val loginResult = it ?: return@Observer

            loading.visibility = View.GONE
            if (loginResult.error != null) {
                showLoginFailed(loginResult.error)
            }
            if (loginResult.success != null) {
                updateUiWithUser(loginResult.success)
                setResult(Activity.RESULT_OK)
                intent = Intent(this, RegisterInformationActivity::class.java)
                intent.putExtra("USER", registerViewModel.getUserFromRepository())
                startActivity(intent)
                //Complete and destroy login activity once successful
                finish()
            }

        })

```
### Memento Pattern

Memento is a behavioral design pattern that lets us save and restore previous states of an object without revealing the dtails of its implementation. Within the Lost and Found app, it was implemented to keep track of the previous changes done within the Announcement Creation Activity.

The activity has a method for creating an AnnouncementSnapshot and setting information.

```
fun createSnapshot():AnnouncementSnapshot{
        var c = if(typeChoice.checkedRadioButtonId == R.id.ann_type_lost) "lost" else "found"
        return AnnouncementSnapshot(this, title.text.toString(), details.text.toString(), c, "wallet")
    }

    fun restore( title:String,  details:String,  type :String,  tag :String){
        this.title.setText(title)
        this.details.setText(details)
        if(type == "lost") typeChoice.check(R.id.ann_type_lost) else typeChoice.check(R.id.ann_type_found)


    }
```
while the Announcement Snapshot saves the information and has the ability to restore the previous state of the Announcement
```
data class AnnouncementSnapshot(var context : CreateAnnouncementActivity, var title:String, var details:String, var type :String, var tag :String ) :Snapshot {
    override fun restore() {
        context.restore(title, details, type, tag)
    }
}

```
with History keeping track of the latest snapshot and having the task of saving the current state periodically.

```
class History(var context: CreateAnnouncementActivity){
    private lateinit var backup : AnnouncementSnapshot

    init {
       GlobalScope.launch (Dispatchers.Default){
           while (!context.isFinishing){
            delay(10000)
               makeBackup()
           }
        }

    }

    fun  makeBackup(){
        backup = context.createSnapshot()
    }

    fun undo(){
        backup.restore()
    }

}

```


## Conclusions / Screenshots / Results

In conclusion, Behavioral Design patterns let us perform effective communication and assignment of responsibilities between objects, allowing us to implement needed functionality easily. 
