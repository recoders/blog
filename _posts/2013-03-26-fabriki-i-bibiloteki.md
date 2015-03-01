---
title: Фабрики и бибилотеки
author: theambient
layout: post
permalink: /2013/fabriki-i-bibiloteki/
Hide SexyBookmarks:
  - 0
Hide OgTags:
  - 0
categories:
  - Общее
---
NB: скучный и технический пост о землях Страуструповых.

Потребовалось засунуть фабрику (см. GoF [1]) в статическую библиотеку и это породило \*удивительные\* эффекты, о которых по крайней мере я ранее не знал.

<!--more-->

Итак, имеем фабрику в следующем виде (ее устройство не обсуждаем, несмотря на то, что она самописная, ее структура стандартная с небольшими плюшками в виде параметризуемого набора аргументов конструктора, смотри Александреску [2]):

Factory.h

<pre>template&lt;typename Base, typename… Args>
class NewCreatorBase
{
public:
    virtual Base * operator()(Args… args) = 0;
    virtual ~NewCreatorBase(){}
};


template&lt;typename Base, typename Derived, typename… Args >
class NewCreatorImpl : public NewCreatorBase&lt;Base, Args…>
{
public:
    Derived * operator()(Args… args) override
    {
        return new Derived(args…);
    }
};

template&lt;typename Base, typename… Args>
class NewCreatorHolder
{
    std::unique_ptr&lt;NewCreatorBase&lt;Base, Args…> > _pimpl; // NOT actual _pimpl!!!
public:
    template&lt;typename Derived>
    explicit NewCreatorHolder(boost::mpl::identity&lt;Derived>  )
    {
        _pimpl.reset(new NewCreatorImpl&lt;Base, Derived, Args…> );
    }

    Base * operator()(Args… args)
    {
        return _pimpl->operator()( args… );
    }
};

template&lt;typename Product, typename Identifier, typename Creator = NewCreatorHolder&lt;Product> >
class Factory
{
    typedef Factory&lt;Product, Identifier,Creator> ThisType;

    std::map&lt;Identifier,Creator> _creators;

public:

    typedef Identifier    IdentifierType;
    typedef Product        ProductType;
    typedef Creator        CreatorType;

    int register_type( Identifier id, Creator creator )
    {
        auto r = _creators.insert( std::make_pair(id, std::move(creator) ) );

        if( ! r.second )
        {
            std::ostringstream ost;
            ost &lt;&lt; "Factory::register_type: id: " &lt;&lt; id &lt;&lt; " already registered";
        
            throw std::logic_error( ost.str() );
        }

        return 1;
    }


    static ThisType * instance()
    {
        static ThisType static_instance;
    
        return &#038;static_instance;
    }


    bool find(Identifier id)
    {
        return _creators.count(id) > 0;
    }

    std::vector&lt;Identifier> list_all_registered_id() const
    {
        std::vector&lt;Identifier> ret;
    
        for( auto &#038; x : _creators )
        {
            ret.push_back( x.first );
        }
    
        return ret;
    }

    template&lt;typename… Args>
    std::unique_ptr&lt;Product> create(Identifier id, Args… args)
    {
        auto it = _creators.find( id );
        if( it == _creators.end() )
        {
            std::ostringstream ost;
            ost &lt;&lt; "Factory::create: id: " &lt;&lt; id &lt;&lt; " has not been registered";

            throw std::runtime_error( ost.str() );
        }
    
        return std::unique_ptr&lt;Product>( it->second(args…) );
    }


};

template&lt;typename ConcreteProduct, typename Factory>
class FactoryRegistrar
{
public:
    FactoryRegistrar(const typename Factory::IdentifierType &#038; id )
    {
        // std::cout &lt;&lt; "FactoryRegistrar: " &lt;&lt; __PRETTY_FUNCTION__ &lt;&lt; std::endl;
        typedef typename Factory::CreatorType T;
        boost::mpl::identity&lt;ConcreteProduct> tag;
        T creator ( tag );
        Factory::instance()->register_type(id, std::move(creator) );            
    }
    
    int get()
    {
        return 1;
    }
    
};
</pre>

Используется фабрика следующим образом. Допустим, нам необходимо создавать классы производные от \`Base\` c помощью фабрики по строковому идентификатору.

Определяем класс \`Base\` и соответствующую фабрику.

Base.h

<pre>class Base
	{
	public:
		Base();
		virtual std::string identify_me() = 0;
	};

	typedef Factory&lt;Base,std::string> BaseFactory;
</pre>

Для регистрации класса \`Derived\` производного от \`Base\` в фабрике в \*\*Derived.cpp\*\* файле необходимо \*определить\* глобально и разумнее всего статически 

Derided.cpp  
`
<pre>static FactoryRegistrar<Derived,BaseFactory> __factory_registrar__;</pre>
<p>`

конструктор которого зарегистрирует \`Derived\` в фабрике. Для включения класса в фабрику теперь достаточно просто включить файл Derived.cpp в сборку (скомпилировать и передать линковщику соответствующий объектный файл). Все кул, что хотели &#8211; то и получили!

Теперь пытаемся вынести все это в \*\*статическую\*\* бибилотеку, которая предоставляет следующий интерфейс:  
&#8211; определение класса \`Base\`  
&#8211; определение фабрики для класса \`Base\`, в которой зарегистрированно сколько-то конкретных производных классов в зависиости от нашей прихоти (с какими опциями собиралсь бибилотека)

Пользователь библиотеки хочет просто создать сущность с помощью \`BaseFactory::instance()->create(&#8220;Dervied&#8221;)\`. Если такой класс не был зарегистрирован в библиотеке, то вернется нулевой указатель, иначе указатель на созданный объект.

Проблема заключается в том, что если засунуть все в статическую бибилотеку и собрать проект с ней, то фабрика \*\*всегда\*\* будет возвращать ноль. Это было неприятным сюрпризом, ноги которого растут из правил линковки.

Если передать линковщику объектный файл напрямую, то он обязан включить его в исполняемый файл и, соотвественно, проинициализировать все переменные с статической областью видимости. В случае бибилиотеки, являющейся не более чем собранием объекнтых файлов, линковщик обращается к ней в том и только в том случае, если не может разрешить какую-то ссылку. В этом случае он включает объектный код, соответствующий этой функции и объектный код, ответственный за инициализацию объектов с статической областью видимости. Так как в нашем случае нет неразрешенных ссылок на объектный файл \`Derived.cpp.o\`, то и не происходит инициализации \`FactoryRegistrar\` и регистрации в фабрике.

Для решения этой проблемы необходимо создавать ссылки на объектный файл, путем вызова функции, скажем, \`Derived::dummy()\` определенной в \`Derived.cpp.o\`, чего и хотелось избежать: зависимости фабрики от регистрируемых в ней классов.

Мораль: не помещать фабрики с статические библиотеки.

Библиография:

[1] Э. Гамма, Р. Хелм, Р. Джонсон, Дж. Влиссидес Приемы объектно-ориентированного проектирования. Паттерны проектирования = Design Patterns: Elements of Reusable Object-Oriented Software. — СПб: «Питер», 2007. — С. 366. — ISBN 978-5-469-01136-1

[2] Александреску А. Современное проектирование на С++: Обобщенное программирование и прикладные шаблоны проектирования = Modern C++ Design: Generic Programming and Design Patterns Applied. — С. П.: Вильямс, 2008. — 336 с. — (С++ in Depth). — ISBN 978-5-8459-0351-8

P.S. Мой первый пост на \*re-coders\* на техническую тему и что я обнружваю?  - не работает, использованное
<pre> - кое-как, съедает "эллипсисы" (...), блок для кодеров который не может нормально отображать код - пичалька...</p>