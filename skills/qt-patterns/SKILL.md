---
name: qt-patterns
description: Qt development patterns, Model/View architecture, signal-slot mechanism, QML integration, and cross-platform best practices.
---

# Qt Development Patterns

Comprehensive guide to Qt application development patterns and best practices.

## When to Activate

- Writing Qt/QML applications
- Designing Qt architecture
- Integrating C++ with QML
- Building cross-platform applications
- Reviewing Qt code

## Signal-Slot Mechanism

### Modern Connection Syntax (Qt5+)

```cpp
// Type-safe, compile-time checked
connect(sender, &Sender::valueChanged,
        receiver, &Receiver::setValue);

// With lambda
connect(button, &QPushButton::clicked, this, [this]() {
    handleButtonClick();
});

// Overloaded signals - use qOverload
connect(spinBox, qOverload<int>(&QSpinBox::valueChanged),
        this, &MyWidget::onValueChanged);

// Qt6: simplified overload syntax
connect(spinBox, &QSpinBox::valueChanged,
        this, &MyWidget::onValueChanged);  // int version
```

### Connection Types

```cpp
// Auto connection (default) - Qt determines best type
connect(sender, &Sender::signal, receiver, &Receiver::slot);

// Direct connection - synchronous, same thread
connect(sender, &Sender::signal, receiver, &Receiver::slot,
        Qt::DirectConnection);

// Queued connection - async, cross-thread safe
connect(worker, &Worker::resultReady,
        this, &MainWindow::handleResult,
        Qt::QueuedConnection);

// Unique connection - prevent duplicates
connect(sender, &Sender::signal, receiver, &Receiver::slot,
        Qt::UniqueConnection);

// Blocking queued - wait for slot to complete
connect(sender, &Sender::signal, receiver, &Receiver::slot,
        Qt::BlockingQueuedConnection);  // Careful: can deadlock!
```

### Custom Signals and Slots

```cpp
class Counter : public QObject {
    Q_OBJECT
    Q_PROPERTY(int value READ value WRITE setValue NOTIFY valueChanged)
    
public:
    int value() const { return m_value; }
    
public slots:
    void setValue(int value) {
        if (m_value != value) {
            m_value = value;
            emit valueChanged(m_value);
        }
    }
    
    void increment() {
        setValue(m_value + 1);
    }
    
signals:
    void valueChanged(int newValue);
    
private:
    int m_value = 0;
};
```

## Memory Management

### Parent-Child Ownership

```cpp
// Parent takes ownership - automatic deletion
QWidget* parent = new QWidget();
QLabel* label = new QLabel("Text", parent);  // parent owns label
QPushButton* button = new QPushButton("Click", parent);

delete parent;  // Also deletes label and button

// Reparenting
label->setParent(nullptr);  // Now you must manage label's lifetime
label->setParent(newParent);  // newParent now owns label
```

### Smart Pointers with Qt

```cpp
// QScopedPointer - similar to unique_ptr
QScopedPointer<QFile> file(new QFile("data.txt"));
if (file->open(QIODevice::ReadOnly)) {
    // Use file
}  // Automatically deleted

// QSharedPointer - reference counted
QSharedPointer<Data> data = QSharedPointer<Data>::create();
QWeakPointer<Data> weakRef = data;

// For QObject with parent, prefer raw pointers
// Parent manages lifetime
QLabel* label = new QLabel(this);  // 'this' is parent
```

### Preventing Memory Leaks

```cpp
// Bad: Leak if never shown or closed
void showDialog() {
    auto* dialog = new MyDialog();
    dialog->show();  // If user closes, memory leaks
}

// Good: Delete on close
void showDialog() {
    auto* dialog = new MyDialog(this);
    dialog->setAttribute(Qt::WA_DeleteOnClose);
    dialog->show();
}

// Good: Modal dialog with local scope
void showDialog() {
    MyDialog dialog(this);
    if (dialog.exec() == QDialog::Accepted) {
        // Handle acceptance
    }
}  // dialog automatically destroyed
```

## Model/View Architecture

### Custom Model Implementation

```cpp
class ContactModel : public QAbstractTableModel {
    Q_OBJECT
    
public:
    enum Column { Name, Email, Phone, ColumnCount };
    
    explicit ContactModel(QObject* parent = nullptr)
        : QAbstractTableModel(parent) {}
    
    int rowCount(const QModelIndex& parent = QModelIndex()) const override {
        if (parent.isValid()) return 0;
        return m_contacts.size();
    }
    
    int columnCount(const QModelIndex& parent = QModelIndex()) const override {
        if (parent.isValid()) return 0;
        return ColumnCount;
    }
    
    QVariant data(const QModelIndex& index, int role = Qt::DisplayRole) const override {
        if (!index.isValid() || index.row() >= m_contacts.size())
            return QVariant();
        
        const Contact& contact = m_contacts[index.row()];
        
        if (role == Qt::DisplayRole || role == Qt::EditRole) {
            switch (index.column()) {
                case Name: return contact.name;
                case Email: return contact.email;
                case Phone: return contact.phone;
            }
        }
        return QVariant();
    }
    
    bool setData(const QModelIndex& index, const QVariant& value, 
                 int role = Qt::EditRole) override {
        if (!index.isValid() || role != Qt::EditRole)
            return false;
        
        Contact& contact = m_contacts[index.row()];
        switch (index.column()) {
            case Name: contact.name = value.toString(); break;
            case Email: contact.email = value.toString(); break;
            case Phone: contact.phone = value.toString(); break;
            default: return false;
        }
        
        emit dataChanged(index, index, {role});
        return true;
    }
    
    Qt::ItemFlags flags(const QModelIndex& index) const override {
        if (!index.isValid()) return Qt::NoItemFlags;
        return Qt::ItemIsEditable | QAbstractTableModel::flags(index);
    }
    
    QVariant headerData(int section, Qt::Orientation orientation,
                        int role = Qt::DisplayRole) const override {
        if (role != Qt::DisplayRole || orientation != Qt::Horizontal)
            return QVariant();
        
        switch (section) {
            case Name: return tr("Name");
            case Email: return tr("Email");
            case Phone: return tr("Phone");
        }
        return QVariant();
    }
    
    // Custom methods for manipulation
    void addContact(const Contact& contact) {
        beginInsertRows(QModelIndex(), m_contacts.size(), m_contacts.size());
        m_contacts.append(contact);
        endInsertRows();
    }
    
    void removeContact(int row) {
        if (row < 0 || row >= m_contacts.size()) return;
        beginRemoveRows(QModelIndex(), row, row);
        m_contacts.removeAt(row);
        endRemoveRows();
    }
    
private:
    QVector<Contact> m_contacts;
};
```

### Proxy Models

```cpp
// Sorting and filtering
class ContactFilterModel : public QSortFilterProxyModel {
    Q_OBJECT
    
public:
    explicit ContactFilterModel(QObject* parent = nullptr)
        : QSortFilterProxyModel(parent) {}
    
    void setFilterText(const QString& text) {
        m_filterText = text;
        invalidateFilter();
    }
    
protected:
    bool filterAcceptsRow(int sourceRow, 
                          const QModelIndex& sourceParent) const override {
        if (m_filterText.isEmpty()) return true;
        
        QModelIndex nameIndex = sourceModel()->index(
            sourceRow, ContactModel::Name, sourceParent);
        QString name = sourceModel()->data(nameIndex).toString();
        
        return name.contains(m_filterText, Qt::CaseInsensitive);
    }
    
private:
    QString m_filterText;
};

// Usage
auto* filterModel = new ContactFilterModel(this);
filterModel->setSourceModel(contactModel);
tableView->setModel(filterModel);
```

## Threading Patterns

### Worker Object Pattern

```cpp
// Worker class
class DataProcessor : public QObject {
    Q_OBJECT
    
public slots:
    void process(const QString& data) {
        // Heavy processing here
        QString result = doHeavyWork(data);
        emit finished(result);
    }
    
signals:
    void finished(const QString& result);
    void progress(int percent);
};

// Setup in main thread
void MainWindow::startProcessing() {
    QThread* thread = new QThread(this);
    DataProcessor* processor = new DataProcessor();
    processor->moveToThread(thread);
    
    // Connect signals
    connect(thread, &QThread::started, 
            processor, [processor]() { processor->process("data"); });
    connect(processor, &DataProcessor::finished,
            this, &MainWindow::handleResult);
    connect(processor, &DataProcessor::finished,
            thread, &QThread::quit);
    connect(thread, &QThread::finished,
            processor, &QObject::deleteLater);
    connect(thread, &QThread::finished,
            thread, &QObject::deleteLater);
    
    thread->start();
}
```

### QtConcurrent for Simple Parallelism

```cpp
#include <QtConcurrent>

// Run function in thread pool
QFuture<Result> future = QtConcurrent::run([this]() {
    return computeHeavyResult();
});

// Monitor with QFutureWatcher
auto* watcher = new QFutureWatcher<Result>(this);
connect(watcher, &QFutureWatcher<Result>::finished, this, [this, watcher]() {
    Result result = watcher->result();
    handleResult(result);
    watcher->deleteLater();
});
watcher->setFuture(future);

// Parallel map
QList<Image> images = loadImages();
QFuture<void> future = QtConcurrent::map(images, processImage);

// Parallel filter
QFuture<Image> filtered = QtConcurrent::filtered(images, 
    [](const Image& img) { return img.isValid(); });
```

## QML Integration

### Exposing C++ to QML

```cpp
// Registering types
// In main.cpp or plugin
qmlRegisterType<MyClass>("MyApp", 1, 0, "MyClass");

// Singleton
qmlRegisterSingletonType<Settings>("MyApp", 1, 0, "Settings",
    [](QQmlEngine*, QJSEngine*) -> QObject* {
        return new Settings();
    });

// Context property (for existing instances)
engine.rootContext()->setContextProperty("backend", &backend);

// C++ class with QML properties
class User : public QObject {
    Q_OBJECT
    Q_PROPERTY(QString name READ name WRITE setName NOTIFY nameChanged)
    Q_PROPERTY(int age READ age WRITE setAge NOTIFY ageChanged)
    QML_ELEMENT  // Qt6: Auto-register
    
public:
    QString name() const { return m_name; }
    void setName(const QString& name) {
        if (m_name != name) {
            m_name = name;
            emit nameChanged();
        }
    }
    
    int age() const { return m_age; }
    void setAge(int age) {
        if (m_age != age) {
            m_age = age;
            emit ageChanged();
        }
    }
    
    Q_INVOKABLE void updateProfile() {
        // Called from QML
    }
    
signals:
    void nameChanged();
    void ageChanged();
    
private:
    QString m_name;
    int m_age = 0;
};
```

### QML Usage

```qml
import QtQuick 2.15
import QtQuick.Controls 2.15
import MyApp 1.0

ApplicationWindow {
    width: 400
    height: 300
    
    MyClass {
        id: myObject
        name: "Example"
        onNameChanged: console.log("Name changed to:", name)
    }
    
    Column {
        TextField {
            text: myObject.name
            onTextChanged: myObject.name = text
        }
        
        Button {
            text: "Update"
            onClicked: myObject.updateProfile()
        }
    }
}
```

### C++ Model in QML

```cpp
class MessageModel : public QAbstractListModel {
    Q_OBJECT
    QML_ELEMENT
    
public:
    enum Roles {
        TextRole = Qt::UserRole + 1,
        SenderRole,
        TimestampRole
    };
    
    QHash<int, QByteArray> roleNames() const override {
        return {
            {TextRole, "text"},
            {SenderRole, "sender"},
            {TimestampRole, "timestamp"}
        };
    }
    
    int rowCount(const QModelIndex& parent = QModelIndex()) const override {
        return m_messages.size();
    }
    
    QVariant data(const QModelIndex& index, int role) const override {
        if (!index.isValid()) return QVariant();
        
        const Message& msg = m_messages[index.row()];
        switch (role) {
            case TextRole: return msg.text;
            case SenderRole: return msg.sender;
            case TimestampRole: return msg.timestamp;
        }
        return QVariant();
    }
    
    Q_INVOKABLE void addMessage(const QString& text, const QString& sender) {
        beginInsertRows(QModelIndex(), m_messages.size(), m_messages.size());
        m_messages.append({text, sender, QDateTime::currentDateTime()});
        endInsertRows();
    }
    
private:
    QVector<Message> m_messages;
};
```

```qml
ListView {
    model: MessageModel {}
    delegate: Row {
        Text { text: model.sender + ": " }
        Text { text: model.text }
        Text { text: model.timestamp.toLocaleTimeString() }
    }
}
```

## Cross-Platform Best Practices

### Platform Detection

```cpp
#ifdef Q_OS_WIN
    // Windows-specific code
#elif defined(Q_OS_MACOS)
    // macOS-specific code
#elif defined(Q_OS_LINUX)
    // Linux-specific code
#elif defined(Q_OS_ANDROID)
    // Android-specific code
#elif defined(Q_OS_IOS)
    // iOS-specific code
#endif

// Runtime check
if (QSysInfo::productType() == "windows") {
    // Windows runtime code
}
```

### Path Handling

```cpp
// Use Qt path utilities
QString configPath = QStandardPaths::writableLocation(
    QStandardPaths::ConfigLocation);
QString dataPath = QStandardPaths::writableLocation(
    QStandardPaths::AppDataLocation);

// File paths with QDir
QDir appDir(QCoreApplication::applicationDirPath());
QString pluginPath = appDir.filePath("plugins");

// Ensure directory separators are correct
QString path = QDir::toNativeSeparators("/path/to/file");
```

### High DPI Support

```cpp
// In main.cpp before QApplication
QCoreApplication::setAttribute(Qt::AA_EnableHighDpiScaling);
QCoreApplication::setAttribute(Qt::AA_UseHighDpiPixmaps);

// Qt6: automatic, but can configure
QGuiApplication::setHighDpiScaleFactorRoundingPolicy(
    Qt::HighDpiScaleFactorRoundingPolicy::PassThrough);
```

## Settings and Configuration

```cpp
// Application-wide settings
QCoreApplication::setOrganizationName("MyCompany");
QCoreApplication::setApplicationName("MyApp");

// Use QSettings
QSettings settings;
settings.setValue("window/size", size());
settings.setValue("window/pos", pos());
settings.setValue("user/name", userName);

// Read settings
resize(settings.value("window/size", QSize(800, 600)).toSize());
move(settings.value("window/pos", QPoint(100, 100)).toPoint());

// Grouped settings
settings.beginGroup("network");
settings.setValue("host", "localhost");
settings.setValue("port", 8080);
settings.endGroup();
```

## Error Handling Patterns

```cpp
// Check operations
QFile file("data.txt");
if (!file.open(QIODevice::ReadOnly)) {
    qWarning() << "Failed to open file:" << file.errorString();
    return;
}

// Network error handling
connect(reply, &QNetworkReply::errorOccurred,
        this, [](QNetworkReply::NetworkError error) {
    qWarning() << "Network error:" << error;
});

// Database errors
QSqlQuery query;
if (!query.exec("SELECT * FROM users")) {
    qWarning() << "Query failed:" << query.lastError().text();
}

// Use qDebug, qInfo, qWarning, qCritical, qFatal
qDebug() << "Debug message";
qInfo() << "Info message";
qWarning() << "Warning message";
qCritical() << "Critical error";
// qFatal() << "Fatal error";  // Terminates app
```

## Resource System

```cpp
// resources.qrc
/*
<RCC>
    <qresource prefix="/images">
        <file>logo.png</file>
        <file>icons/save.svg</file>
    </qresource>
    <qresource prefix="/data">
        <file>config.json</file>
    </qresource>
</RCC>
*/

// Usage
QPixmap logo(":/images/logo.png");
QIcon saveIcon(":/images/icons/save.svg");
QFile config(":/data/config.json");
```

## Best Practices Summary

| Pattern | When to Use |
|---------|-------------|
| Signal-Slot | Loose coupling between components |
| Model/View | Display and edit collections |
| Worker Thread | CPU-intensive operations |
| QtConcurrent | Simple parallel operations |
| QML | Fluid UI, mobile apps |
| QSettings | Persistent configuration |
| Resource System | Embedded assets |

### Anti-Patterns to Avoid

- Using old-style `SIGNAL()/SLOT()` macros
- Blocking the GUI thread
- Creating GUI objects from non-GUI threads
- Not checking return values of file/network operations
- Hardcoding paths (use QStandardPaths)
- Ignoring Qt's memory management model

**Remember**: Qt provides powerful abstractions. Use them instead of reinventing the wheel. When in doubt, check the Qt documentation.
