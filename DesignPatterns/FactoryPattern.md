# 1.工厂与简单工厂模式

## 1.1 简单工厂
    
    以Telephony中的State抽象类为例子：

    定义抽象产品类：

    class IState : public RefBase {
    public:
        virtual void enter() = 0;
        virtual void exit() = 0;
        virtual int processMessage(const sp<Message> msg) = 0;
    }

    具体的State:

    class RilInitState : public IState {
    public:
        RilInitState() {}
        ~RilInitState() {}

        void enter() {

        }

        void exit() {
            
        }

        int processMessage(const sp<Message>& msg) {
            return 1;
        }
    };

    class RilPowerOnState : public IState {
    public:
        RilPowerOnState() {}
        ~RilPowerOnState() {}

        void enter() {

        }

        void exit() {
            
        }

        int processMessage(const sp<Message>& msg) {
            return 1;
        }
    };

    简单工厂类：

    通过简单的 if/else 判断创建对应的State.

    class StateFactory {
    public :
        static sp<IState> CreateState(sp<StateMachine> hsm, std::string name, std::string parent = "") {
            std::map<std::string, int> stateMap = {
                {"RILINITSTATE", 0},
                {"RILPOWERONSTATE", 1}
            };
            if (stateMap.count(name) != 1) {
                printf("unsupport state!\n");
            }
            sp<IState> state;
            switch(stateMap[name]) {
                case 0:
                    state = new RilInitState();
                    break;
                case 1:
                    state = new RilPowerOnState();
                    break;
                default:
                    printf("can't make state!\n");
            }

            // hsm->addState(state, name, parent);

            return state;
        }
    };

    优点：
    1.工厂类中包括必要的逻辑判断，可以决定什么时候创建什么实例。客户端不再考虑创建产品对象的职责
    2.客户端只需要知道参数即可
    2.可以搭配配置文件，不修改客户端代码的情况下更换与添加新产品类

    缺点：
    1.工厂职责重大，出现异常有极大的影响
    2.系统拓展困难，一旦增加新产品则需要修改工厂逻辑，违反了开闭原则
    3.使用了static工厂方法，无法被继承

## 1.2工厂方法模式

    原理就是定义一个创建对象的接口，让其子类决定实例化哪一个工厂类，一个工厂类创建一个实例，工厂模式使其创建过程延迟到子类进行。

    也就是：

    class IState : public RefBase {
    public:
        virtual void enter() = 0;
        virtual void exit() = 0;
        virtual int processMessage(const sp<Message> msg) = 0;
    }

    class RilInitState : public IState {
    public:
        RilInitState() {}
        ~RilInitState() {}

        void enter() {

        }

        void exit() {
            
        }

        int processMessage(const sp<Message>& msg) {
            return 1;
        }
    };

    class RilPowerOnState : public IState {
    public:
        RilPowerOnState() {}
        ~RilPowerOnState() {}

        void enter() {

        }

        void exit() {
            
        }

        int processMessage(const sp<Message>& msg) {
            return 1;
        }
    };

    class IFactory : public RefBase {
    public:
        virtual sp<IState> createState() = 0;
    };

    class InitFactory : public IFactory {
    public:
        virtual sp<IState> createState() {
            return new RilInitState();
        }
    };

    class PowerOnFactory : public IFactory {
    public:
        virtual sp<IState> createState() {
            return new RilPowerOnState();
        }
    };

    优点：
    1.符合开闭原则
    2.客户端无须记住具体类名
    3.对象的创建与使用分离
    4.可拓展性高，无须修改接口与原有类

    缺点：
    1.类个数增加太多
    2.系统抽象性增加

## 1.3 抽象工厂模式

    作用：提供一个创建一系列相关或者相互依赖项的对象的接口，无须指定具体类

    主要解决的问题：主要解决接口选择的问题。

    例子：假设RIL目前的设计需要针对不同的modem （高通，展讯，华为）进行开发；对于modem上电，以及通过串口发送AT指令的方式不同（高通是Qril,展讯与华为都是通过AT指令发送给串口）。那么对于不同的modem,除了通用的拨号状态管理之外，还需要有PowerController，ATChannel模块。

    powerController，AtChannel互相关联甚至依赖（为统一品牌Modem服务）。此处即可以使用抽象工厂模式来生成一套实例。

    PowerController类与实体类：

    class IPower : public RefBase {
    public:
        virtual void up() = 0;
        virtual void down() = 0;
        virtual void reset() = 0;
    };

    class QualComPc : public IPower {
    public:
        QualComPc() { printf("QualComPc create\n"); }
        ~QualComPc() {}

        virtual void up() {

        }

        virtual void down() {
            
        }

        virtual void reset() {

        }
    };

    class HuaWeiPc : public IPower {
    public:
        HuaWeiPc() { printf("HuaWeiPc create\n"); }
        ~HuaWeiPc() {}

        virtual void up() {

        }

        virtual void down() {
            
        }

        virtual void reset() {
            
        }
    };

    //AtChannel抽象类与实体类：

    class IAtChannel : public RefBase {
    public:
        virtual void setup() = 0;
        virtual void write() = 0;
        virtual void read() = 0;
    };

    class QualComAtChannel : public IAtChannel {
    public:
        QualComAtChannel() { printf("QualComAtChannel create\n"); }
        ~QualComAtChannel() {}

        virtual void setup() {

        }

        virtual void write() {
            
        }

        virtual void read() {

        }
    };

    class HuaWeiAtChannel : public IAtChannel {
    public:
        HuaWeiAtChannel() { printf("HuaWeiAtChannel create\n"); }
        ~HuaWeiAtChannel() {}

        virtual void setup() {

        }

        virtual void write() {
            
        }

        virtual void read() {

        }
    };

    //为PowerController与AtChannel创建抽象工厂：

    class IModemFactory : RefBase {
    public:
        virtual sp<IPower> getPowerController() = 0;
        virtual sp<IAtChannel> getAtChannel() = 0;
    };

    class QualComModemFacroty : public IModemFactory {
    public:
        QualComModemFacroty() { printf("QualComModemFacroty create\n");}
        ~QualComModemFacroty() {}
        virtual sp<IPower> getPowerController() {
            return new QualComPc();
        }

        virtual sp<IAtChannel> getAtChannel() {
            return new QualComAtChannel();
        }
    };

    class HuaWeiModemFacroty : public IModemFactory {
    public:
        HuaWeiModemFacroty() { printf("HuaWeiModemFacroty create\n");}
        ~HuaWeiModemFacroty() {}
        virtual sp<IPower> getPowerController() {
            return new HuaWeiPc();
        }

        virtual sp<IAtChannel> getAtChannel() {
            return new HuaWeiAtChannel();
        }
    };




