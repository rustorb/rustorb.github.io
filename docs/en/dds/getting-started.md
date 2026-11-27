# Getting started

A publisher writes five greetings on the topic `hello_world`, and a
subscriber prints them. Both join domain 0, with reliable QoS. Any
publisher below talks with any subscriber, in any language, and with
other DDS implementations that use the same IDL and topic.

[Install](install.md) the package for your language and `rustorb-idl`
first.

## The type

```idl title="HelloWorld.idl"
module HelloWorld {
    struct Greeting {
        unsigned long index;
        string message;
    };
};
```

## Publisher and subscriber

=== "Python"

    ```sh
    rustorb-idl -bpython -Wbdds_only HelloWorld.idl   # writes HelloWorld_dds.py
    ```

    ```python title="pub.py"
    import time
    import rustorb_dds as dds
    from HelloWorld_dds import HelloWorld

    participant = dds.DomainParticipant(0)
    topic = participant.create_topic("hello_world", HelloWorld.Greeting)
    writer = participant.create_publisher().create_datawriter(
        topic, dds.Qos(reliability="reliable"))

    writer.wait_for_subscriptions(1, timeout=10.0)
    time.sleep(0.5)  # for the subscriber to match this writer too
    for i in range(1, 6):
        writer.write(HelloWorld.Greeting(i, "Hello, world!"))
        time.sleep(0.5)
    writer.wait_for_acknowledgments(5.0)
    ```

    ```python title="sub.py"
    import rustorb_dds as dds
    from HelloWorld_dds import HelloWorld

    participant = dds.DomainParticipant(0)
    topic = participant.create_topic("hello_world", HelloWorld.Greeting)
    reader = participant.create_subscriber().create_datareader(
        topic, dds.Qos(reliability="reliable"))

    while True:
        if reader.wait_for_data(10.0):
            for s in reader.take():
                if s.data is not None:
                    print(s.data.index, s.data.message)
    ```

    ```sh
    python sub.py      # in one terminal
    python pub.py      # in another
    ```

    Instead of IDL, a type can also be declared in Python with
    `@dds.struct("HelloWorld::Greeting")` on a class with annotated members
    (see the [package's README](https://github.com/rustorb/rustorb-dds/tree/main/python)).

=== "Rust"

    Put `HelloWorld.idl` in `idl/`, and the [dependencies](install.md#rust)
    in `Cargo.toml`.

    ```rust title="build.rs"
    fn main() {
        rustorb_codegen::Config::new()
            .dds_only()
            .compile(&["idl/HelloWorld.idl"]) // $OUT_DIR/hello_world.rs
            .unwrap();
    }
    ```

    ```rust title="src/main.rs"
    use std::time::Duration;

    use rustorb_dds::{DataReaderQos, DataWriterQos, DomainParticipant, Reliability};

    include!(concat!(env!("OUT_DIR"), "/hello_world.rs"));
    use hello_world::Greeting;

    fn main() -> Result<(), Box<dyn std::error::Error>> {
        let participant = DomainParticipant::new(0)?;
        let topic = participant.create_topic::<Greeting>("hello_world")?;

        if std::env::args().nth(1).as_deref() == Some("sub") {
            let qos = DataReaderQos {
                reliability: Reliability::reliable(),
                ..DataReaderQos::default()
            };
            let reader = participant
                .create_subscriber()?
                .create_datareader_with_qos(&topic, qos)?;
            loop {
                if reader.wait_for_data(Duration::from_secs(10)) {
                    for sample in reader.take() {
                        if let Some(g) = sample.data {
                            println!("{} {}", g.index, g.message);
                        }
                    }
                }
            }
        } else {
            let qos = DataWriterQos {
                reliability: Reliability::reliable(),
                ..DataWriterQos::default()
            };
            let writer = participant
                .create_publisher()?
                .create_datawriter_with_qos(&topic, qos)?;
            writer.wait_for_subscriptions(1, Duration::from_secs(10));
            // For the subscriber to match this writer too.
            std::thread::sleep(Duration::from_millis(500));
            for index in 1..=5 {
                writer.write(&Greeting { index, message: "Hello, world!".into() })?;
                std::thread::sleep(Duration::from_millis(500));
            }
            writer.wait_for_acknowledgments(Duration::from_secs(5))?;
        }
        Ok(())
    }
    ```

    ```sh
    cargo run -- sub   # in one terminal
    cargo run -- pub   # in another
    ```

=== "C++"

    ```sh
    rustorb-idl -bc++11 -Wbdds_only HelloWorld.idl   # HelloWorld.hh, HelloWorld_dds.hh
    ```

    ```cpp title="pub.cc"
    #include "HelloWorld_dds.hh"
    #include <dds/dds.hpp>

    #include <chrono>
    #include <iostream>
    #include <thread>

    int main() {
      dds::domain::DomainParticipant dp(0);
      dds::topic::Topic<HelloWorld::Greeting> topic(dp, "hello_world");
      dds::pub::Publisher pub(dp);
      dds::pub::qos::DataWriterQos qos = pub.default_datawriter_qos();
      qos << dds::core::policy::Reliability::Reliable();
      dds::pub::DataWriter<HelloWorld::Greeting> writer(pub, topic, qos);

      while (!writer.wait_for_subscriptions(1, dds::core::Duration(1))) {
      }
      // For the subscriber to match this writer too.
      std::this_thread::sleep_for(std::chrono::milliseconds(500));
      for (uint32_t i = 1; i <= 5; i++) {
        writer << HelloWorld::Greeting(i, "Hello, world!");
        std::this_thread::sleep_for(std::chrono::milliseconds(500));
      }
      try {
        writer.wait_for_acknowledgments(dds::core::Duration(5));
      } catch (const dds::core::TimeoutError&) {
        std::cerr << "not all acknowledged" << std::endl;
        return 1;
      }
    }
    ```

    ```cpp title="sub.cc"
    #include "HelloWorld_dds.hh"
    #include <dds/dds.hpp>

    #include <iostream>

    int main() {
      dds::domain::DomainParticipant dp(0);
      dds::topic::Topic<HelloWorld::Greeting> topic(dp, "hello_world");
      dds::sub::Subscriber sub(dp);
      dds::sub::qos::DataReaderQos qos = sub.default_datareader_qos();
      qos << dds::core::policy::Reliability::Reliable();
      dds::sub::DataReader<HelloWorld::Greeting> reader(sub, topic, qos);

      dds::core::cond::StatusCondition status(reader);
      status.enabled_statuses(dds::core::status::StatusMask::data_available());
      dds::core::cond::WaitSet waitset;
      waitset += status;
      for (;;) {
        waitset.wait(dds::core::Duration::infinite());
        for (const auto& s : reader.take()) {
          if (s.info().valid())
            std::cout << s.data().index() << " " << s.data().message() << std::endl;
        }
      }
    }
    ```

    ```cmake title="CMakeLists.txt"
    cmake_minimum_required(VERSION 3.16)
    project(hello CXX)
    set(CMAKE_CXX_STANDARD 17)
    find_package(rustorb_dds REQUIRED)
    add_executable(pub pub.cc)
    add_executable(sub sub.cc)
    target_link_libraries(pub PRIVATE rustorb_dds::rustorb_dds)
    target_link_libraries(sub PRIVATE rustorb_dds::rustorb_dds)
    ```

=== "Java"

    ```sh
    rustorb-idl -bjava -Wbdds_only HelloWorld.idl   # HelloWorld/Greeting.java, HelloWorld/GreetingCodec.java
    ```

    ```java title="Pub.java"
    import HelloWorld.Greeting;
    import java.util.concurrent.TimeUnit;
    import org.omg.dds.core.Duration;
    import org.omg.dds.core.ServiceEnvironment;
    import org.omg.dds.core.policy.PolicyFactory;
    import org.omg.dds.domain.DomainParticipant;
    import org.omg.dds.pub.DataWriter;
    import org.omg.dds.pub.Publisher;
    import org.omg.dds.topic.Topic;
    import org.rustorb.dds.Rustorb;

    public class Pub {
        public static void main(String[] args) throws Exception {
            ServiceEnvironment env = ServiceEnvironment.createInstance(Pub.class.getClassLoader());
            PolicyFactory policy = PolicyFactory.getPolicyFactory(env);
            DomainParticipant dp = env.getSPI().getParticipantFactory().createParticipant(0);
            Topic<Greeting> topic = dp.createTopic("hello_world", Greeting.class);
            Publisher pub = dp.createPublisher();
            DataWriter<Greeting> writer = pub.createDataWriter(topic,
                    pub.getDefaultDataWriterQos().withPolicy(policy.Reliability().withReliable()));
            Rustorb.waitForSubscriptions(writer, 1, Duration.newDuration(10, TimeUnit.SECONDS, env));
            Thread.sleep(500); // for the subscriber to match this writer too
            for (int i = 1; i <= 5; i++) {
                writer.write(new Greeting(i, "Hello, world!"));
                Thread.sleep(500);
            }
            writer.waitForAcknowledgments(5, TimeUnit.SECONDS);
            dp.close();
        }
    }
    ```

    Run it with `java --enable-native-access=ALL-UNNAMED
    -Drustorb.dds.ffi.library=/path/to/librustorb_dds_ffi.so ...`. The
    subscriber side, with a WaitSet, is
    [HelloWorldExample.java](https://github.com/rustorb/rustorb-dds/blob/main/java/examples/examples/HelloWorldExample.java).

=== "C#"

    ```sh
    rustorb-idl -bcsharp -Wbdds_only HelloWorld.idl   # HelloWorld.cs, HelloWorld_dds.cs
    ```

    ```csharp title="Program.cs"
    using HelloWorld;
    using Omg.Dds.Core;
    using Omg.Dds.Core.Policy;
    using Omg.Dds.Core.Status;
    using Omg.Dds.Domain;
    using Omg.Dds.Pub;
    using Omg.Dds.Sub;
    using Omg.Dds.Topic;

    using DomainParticipant dp = new DomainParticipant(0);
    Topic<Greeting> topic = dp.CreateTopic<Greeting>("hello_world");

    if (args.Length > 0 && args[0] == "sub")
    {
        DataReader<Greeting> reader = dp.CreateSubscriber().CreateDataReader(topic,
            DataReaderQos.Default with { Reliability = Reliability.Reliable() });
        StatusCondition status = reader.StatusCondition;
        status.EnabledStatuses = StatusKind.DataAvailable;
        using var waitset = new WaitSet();
        waitset.AttachCondition(status);
        while (true)
        {
            waitset.Wait(TimeSpan.FromSeconds(60));
            foreach (Sample<Greeting> s in reader.Take())
            {
                if (s.Info.ValidData) Console.WriteLine($"{s.Data!.index} {s.Data.message}");
            }
        }
    }
    else
    {
        DataWriter<Greeting> writer = dp.CreatePublisher().CreateDataWriter(topic,
            DataWriterQos.Default with { Reliability = Reliability.Reliable() });
        writer.WaitForSubscriptions(1, TimeSpan.FromSeconds(10));
        Thread.Sleep(500); // for the subscriber to match this writer too
        for (uint i = 1; i <= 5; i++)
        {
            writer.Write(new Greeting(i, "Hello, world!"));
            Thread.Sleep(500);
        }
        writer.WaitForAcknowledgments(TimeSpan.FromSeconds(5));
    }
    ```

    ```sh
    dotnet new console && dotnet add package Rustorb.Dds
    dotnet run -- sub   # in one terminal
    dotnet run          # in another
    ```

=== "Ruby"

    ```sh
    rustorb-idl -bruby -Wbdds_only HelloWorld.idl   # writes HelloWorld_dds.rb
    ```

    ```ruby title="pub.rb"
    require "rustorb/dds"
    require_relative "HelloWorld_dds"

    dp = DDS::Domain::DomainParticipant.new(0)
    topic = DDS::Topic::Topic.new(dp, "hello_world", HelloWorld::Greeting)
    qos = DDS::Pub::DataWriterQos.new << DDS::Core::Policy::Reliability.reliable
    writer = DDS::Pub::DataWriter.new(DDS::Pub::Publisher.new(dp), topic, qos)
    writer.wait_for_subscriptions(1, 10)
    sleep 0.5 # for the subscriber to match this writer too
    (1..5).each do |i|
      writer.write(HelloWorld::Greeting.new(i, "Hello, world!"))
      sleep 0.5
    end
    writer.wait_for_acknowledgments(5)
    dp.close
    ```

    ```ruby title="sub.rb"
    require "rustorb/dds"
    require_relative "HelloWorld_dds"

    dp = DDS::Domain::DomainParticipant.new(0)
    topic = DDS::Topic::Topic.new(dp, "hello_world", HelloWorld::Greeting)
    qos = DDS::Sub::DataReaderQos.new << DDS::Core::Policy::Reliability.reliable
    reader = DDS::Sub::DataReader.new(DDS::Sub::Subscriber.new(dp), topic, qos)
    status = reader.status_condition
    status.enabled_statuses = DDS::Core::Status::StatusMask::DATA_AVAILABLE
    waitset = DDS::Core::Cond::WaitSet.new
    waitset << status
    loop do
      waitset.wait(60)
      reader.take.each { |s| puts "#{s.data.index} #{s.data.message}" if s.valid? }
    end
    ```

=== "C"

    The C ABI is what the C++, Java, .NET and Ruby bindings are built on.
    There is no IDL backend for it: a program builds its types with the
    `rdds_*_builder` functions and writes and reads samples as streams of
    values in the type's order. A complete publisher and subscriber is
    [smoke.c](https://github.com/rustorb/rustorb-dds/blob/main/crates/rustorb-dds-ffi/tests/c/smoke.c).

## With other DDS implementations

The same type and topic name work with other implementations. For
example, with Cyclone DDS's `idlc` output of `HelloWorld.idl`, or with a
ROS 2 node: the repository's `ros2` example talks with
`ros2 run demo_nodes_cpp talker` and `listener` on `std_msgs/msg/String`
(see [Interoperability](interop.md)).

More examples, all with the same command line, are in the repository:
[Rust](https://github.com/rustorb/rustorb-dds/tree/main/crates/rustorb-dds/examples)
(hello world, keyed instances, DynamicData, type evolution, ROS 2 topics
and services, OpenRTM-aist),
[Python](https://github.com/rustorb/rustorb-dds/tree/main/python/examples),
[C++](https://github.com/rustorb/rustorb-dds/tree/main/cpp/examples),
[Java](https://github.com/rustorb/rustorb-dds/tree/main/java/examples/examples),
[.NET](https://github.com/rustorb/rustorb-dds/tree/main/dotnet/examples) and
[Ruby](https://github.com/rustorb/rustorb-dds/tree/main/ruby/examples).
