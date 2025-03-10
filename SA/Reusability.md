# Ideas to Improve Software Reusability
1. Separation of concerns

    Each component should have a single, well-defined responsibility. Also, we should avoid mixing unrelated logic in the same module.

2. Interfaces for defining contracts

    Use interfaces to define contracts among software components, which allows implementation to vary without affecting the dependent modules.

3. Dependency inversion principle

    Traditionally, we have heirarchical software structure. The high level modules build on top of low level modules, which creates a direct dependency.

    ```cpp
    class ModbusDriver {
    public:
        void readData() { /* Modbus-specific logic */ }
    };

    class DataAggregator {
    private:
        ModbusDriver modbus;  // Direct dependency on a concrete class
    public:
        void aggregateData() {
            modbus.readData();  // Tightly coupled to Modbus
        }
    };
    ```

    DIP principle suggests that high level modules should not depend on low level modules, instead they should have a shared abstraction(interface). The low level implementation is provided to high level at runtime (generally via dependency injection).

    ```cpp
    // Abstraction (interface)
    class IProtocolDriver {
    public:
        virtual ~IProtocolDriver() = default;
        virtual void readData() = 0;  // Abstract method
    };

    // Concrete implementation for Modbus
    class ModbusDriver : public IProtocolDriver {
    public:
        void readData() override { /* Modbus-specific logic */ }
    };

    // Concrete implementation for DNP3
    class DNP3Driver : public IProtocolDriver {
    public:
        void readData() override { /* DNP3-specific logic */ }
    };

    // High-level module depending on the abstraction
    class DataAggregator {
    private:
        std::unique_ptr<IProtocolDriver> driver;  // Depends on abstraction, not concrete class
    public:
        DataAggregator(std::unique_ptr<IProtocolDriver> driver) : driver(std::move(driver)) {}
        void aggregateData() {
            driver->readData();  // Works with any IProtocolDriver implementation
        }
    };

    // Usage
    int main() {
        // Dependency injection at runtime
        auto modbusDriver = std::make_unique<ModbusDriver>();
        DataAggregator aggregator(std::move(modbusDriver));
        aggregator.aggregateData();

        // Easily switch to DNP3 without changing DataAggregator
        auto dnp3Driver = std::make_unique<DNP3Driver>();
        DataAggregator aggregator2(std::move(dnp3Driver));
        aggregator2.aggregateData();
    }
    ```

4. Message Bus or Event Driven Architecture

