# Clase-7: Reflection

## Introducción a Reflection

Reflection es la habilidad de un programa de autoexaminarse con el objetivo de encontrar ensamblados (.dll), módulos, o información de tipos en tiempo de ejecución. En otras palabras, a nivel de código vamos a tener clases y objetos, que nos van a permetir referenciar a ensamblados, y a los tipos que se encuentran contenidos.

Se dice que un programa se refleja en sí mismo (de ahí el termino "reflección"), a partir de extraer metadata de sus assemblies y de usar esa metadata para ciertos fines. Ya sea para informarle al usuario o para modificar su comportamiento.

Al usar Reflection en C#, estamos pudiendo obtener la información detallada de un objeto, sus métodos, e incluso crear objetos e invocar sus métodos en tiempo de ejecución, sin haber tenido que realizar una referencia al ensamblado que contiene la clase y a su namespace.

Específicamente lo que nos permite usar Reflection es el namespace ```System.Reflecion```, que contiene clases e interfaces que nos permiten manejar todo lo mencionado anteriormente: ensamblados, tipos, métodos, campos, crear objetos, invocar métodos, etc.

## Estructura de un assembly/ensamblado

Los assemblies contienen módulos, los módulos contienen tipos y los tipos contienen miembros. Reflection provee clases para encapsular estos elementos. Entonces como dijimos posible utilizar reflection para crear dinámicamente instancias de un tipo, obtener el tipo de un objeto existente e invocarle métodos y acceder a sus atributos de manera dinámica. 
 
![alt text](http://www.codeproject.com/KB/cs/DLR/structure.JPG)

## ¿Para qué podría servir?

Supongamos por ejemplo, que necesitamos que nuestra aplicación soporte diferentes tipos de loggers (mecanismos para registrar datos/eventos que van ocurriendo en el flujo del programa). Además, supongamos que hay desarrolladores terceros que nos brindan una .dll externa que escribe información de logger y la envía a un servidor. En ese caso, tenemos dos opciones:

1) Podemos referenciar al ensamblado directamente y llamar a sus métodos (como hemos hecho siempre) 
2) Podemos usar Reflection para cargar el ensamblado y llamar a sus métodos a partir de sus interfaces.

En este caso, si quisieramos que nuestra aplicación sea lo más desacoplada posible, de manera que otros loggers puedan ser agregados (o 'plugged in' -de ahí el nombre plugin-) de forma sencilla y SIN RECOMPILAR la aplicación, es necesario elegir la segunda opción.

Por ejemplo podríamos hacer que el usuario elija (a medida que está usando la aplicación), y descargue la .dll de logger para elegir usarla en la aplicación. La única forma de hacer esto es a partir de Reflection. De esta forma, podemos cargar ensamblados externos a nuestra aplicación, y cargar sus tipos en tiempo de ejecución. 

## Favoreciendo el desacoplamiento

Lo que es importante para lograr el desacoplamiento de tipos externos, es que nuestro código referencie a una Interfaz, que es la que toda .dll externa va a tener que cumplir. Tiene que existir entonces ese contrato previo, de lo contrario, no sería posible saber de antemano qué metodos llamar de las librerías externas que poseen clases para usar loggers.

## Ejemplo en ```C#```

Crearemos un proyecto simple con estas tres clases que se listan a continuación. Tendremos empleados y tipos concretos de empleados (de manera que los empleados concretos cumplen la misma interfaz que los empleados genéricos). Le llamaremos ```EjemplosReflection```

```C#

public abstract class Employee
{
    public string Name { get; set; }
    public string CI { get; set; }
    public Employee()
    {
       Name = "Unnamed";
       CI = "This guy's no documents";
    }
    public Employee(String aName, string aCI)
    {
       Name = aName;
       CI = aCI;
    }
    public override string ToString()
    {
       return string.Format("{0} - {1}", CI, Name);
    }
    public abstract double CalculateSalary();
}

public class MonthlyEmployee : Employee
{
    public double MonthlySalary { get; set; }
    public MonthlyEmployee() { }
    public MonthlyEmployee(string aName, String aCI, double aSalary)
    : base(aName, aCI)
    {
       MonthlySalary = aSalary;
    }
    public override string ToString()
    {
       return string.Format("{0} {1}", "Monnhtly employee", base.ToString());
    }
    public override double CalculateSalary()
    {
       return MonthlySalary;
    }
}
public class HourEmployee : Employee
{
    public double HourValue { get; set; }
    public int HoursWorked {get;set;}
    public HourEmployee() { }
    public HourEmployee(string aName, String aCI, double anHourValue, int someHoursWorked)
    : base(aName, aCI)
    {
       HourValue = anHourValue;
       HoursWorked = someHoursWorked;
    }
    public override string ToString()
    {
       return string.Format("{0} {1}", "Hour Employee", base.ToString());
    }
    public override double CalculateSalary()
    {
       return HoursWorked * HourValue;
    }
}

```
### Inspeccionando el ensamblado

Una vez creadas las clases, vamos a generar el assembly (dll) y a copiarlo a alguna carpeta en el disco. El assembly generado lo podemos encontrar en la carpeta ```EjemplosReflection\bin\Debug```. Recomendamos copiarlo a alguna carpeta de fácil acceso, por ejemplo ```c:\pruebas:```

Ya tenemos el assembly que utilizaremos para las pruebas, por lo que comenzaremos a utilizar
reflection. Vamos a cerrar el proyecto anterior y a crear un proyecto nuevo (independiente del anterior), esta vez del tipo “Aplicación de Consola” a la que llamaremos de la forma que queramos.

Lo primero que probaremos será la capacidad de inspección que ofrece reflection sobre los
assemblies. Para ello, en el método Main agregaremos el siguiente códig. Nota: Se deberán agregar algunos imports.

Primero inspeccionamos el assembly:

```C#
static void Main(string[] args)
{
    // Cargamos el assembly de ejemplo en memoria
    Assembly miAssembly= Assembly.LoadFile(@"c:\Pruebas\EjemplosReflection.dll");
    // Podemos ver que Tipos hay dentro del assembly
    foreach (Type tipo in miAssembly.GetTypes())
    {
       Console.WriteLine(string.Format("Clase: {0}" ,tipo.Name));

       Console.WriteLine("Propiedades");
       foreach (PropertyInfo prop in tipo.GetProperties())
       {
          Console.WriteLine(string.Format("\t{0} : {1}", prop.Name, prop.PropertyType.Name));
       }
       Console.WriteLine("Constructores");
       foreach (ConstructorInfo con in tipo.GetConstructors())
       {
          Console.Write("\tConstructor: ");
          foreach (ParameterInfo param in con.GetParameters())
          {
             Console.Write(string.Format("{0} : {1} ", param.Name,param.ParameterType.Name));
          }
          Console.WriteLine();
       }
       Console.WriteLine();
       Console.WriteLine("Metodos");
       foreach (MethodInfo met in tipo.GetMethods())
       {
          Console.Write(string.Format("\t{0} ", met.Name));
          foreach (ParameterInfo param in met.GetParameters())
          {
             Console.Write(string.Format("{0} : {1} ", param.Name,param.ParameterType.Name));
          }
          Console.WriteLine();
       }
       Console.WriteLine();
    }
    Console.ReadLine();
}
```

Analizar detenidamente la salida.

Acabamos de ver como mediante reflection es posible investigar el contenido de un assembly, obtener su información, como conocer las propiedades, los constructores y los métodos de cada clase. 

### Instanciando tipos dinámicamente 

Como ya hemos mencionado, otra de las pricipales ventajas de reflection es que además de poder conocer información sobre los tipos dentro de un assembly, permite trabajar con ellos de manera dinámica. Para ejemplificarlo, vamos a crear un objeto de la clase HourEmployee utilizando un constructor con parámetros, le vamos a cambiar el valor de una de sus propiedades y luego le invocaremos un método. Todo esto desde nuestra aplicación de consola, que NO tiene una referencia a la dll
con las clases, por lo que todo se hará de manera dinámica. 

Cambiemos el contenido del Main por el siguiente:

```C#

static void Main(string[] args)
{
   // Cargamos el assembly en memoria
   Assembly testAssembly = Assembly.LoadFile(@"c:\Pruebas\EjemplosReflection.dll"); 
   
   // Obtenemos el tipo que representa a Empleado por Hora (HourEmployee)
   Type employeeType = testAssembly.GetType("EjemplosReflection.HourEmployee");
   
   // Creamos una instancia de Empleado por hora
   object employeeInstance = Activator.CreateInstance(employeeType);
   
   // O también podemos crearlo pasandole parámetros
   employeeInstance = Activator.CreateInstance(employeeType, new object[] { "Juan", "1.232.232-3", 25.5, 10 });
   
   // Lo mostramos
   Console.WriteLine(employeeInstance.ToString());
   
   //Invocamos al método CalculateSalary
   MethodInfo met = employeeType.GetMethod("CalculateSalary");
   Console.WriteLine(string.Format("Sueldo: {0}", met.Invoke(employeeInstance, null)));
   
   //También podemos cambiar el valor de las horas trabajadas
   PropertyInfo prop = employeeType.GetProperty("HoursWorked");
   prop.SetValue(employeeInstance, 300, null);
   
   Console.ReadLine();
   
}

```

IMPORTANTE: aquí estamos asumiendo los nombres de los métodos y llamandolos directamente pasandole Strings como parámetros. Esto en un caso más real no sería correcto, ya que primero deberíamos asegurarnos de que el tipo que queremos instanciar cumple con la interfaz (es decir, tiene los métodos), que queremos usar.

Esto se puede hacer preguntando de la siguiente forma:

```C#

typeof(IMyInterface).IsAssignableFrom(typeof(MyType))
typeof(MyType).GetInterfaces().Contains(typeof(IMyInterface))

```
