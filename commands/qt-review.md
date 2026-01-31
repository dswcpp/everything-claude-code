---
description: Qt/QML code review for signal-slot patterns, memory management, thread safety, and Qt best practices. Invokes the qt-reviewer agent.
---

# Qt Code Review

This command invokes the **qt-reviewer** agent for comprehensive Qt-specific code review.

## What This Command Does

1. **Identify Qt Changes**: Find modified `.cpp`, `.hpp`, `.h`, `.qml`, `.ui` files
2. **Check Q_OBJECT Macro**: Verify signal/slot prerequisites
3. **Memory Analysis**: Review QObject parent-child relationships
4. **Thread Safety**: Check cross-thread communication
5. **Qt Best Practices**: Validate Qt idioms and patterns
6. **Generate Report**: Categorize issues by severity

## When to Use

Use `/qt-review` when:
- After writing or modifying Qt code
- Before committing Qt/QML changes
- Reviewing pull requests with Qt code
- Migrating from Qt5 to Qt6
- Learning Qt best practices

## Review Categories

### CRITICAL (Must Fix)
- Missing Q_OBJECT macro for signals/slots
- GUI operations from non-GUI thread
- Memory leaks (no parent, no deleteLater)
- Accessing deleted QObjects
- Cross-thread signal/slot without proper connection

### HIGH (Should Fix)
- Old-style SIGNAL()/SLOT() connections
- Blocking the event loop
- QMutex without QMutexLocker
- Missing model signals (begin/endInsertRows)
- QML ownership issues

### MEDIUM (Consider)
- QString/std::string unnecessary conversions
- Missing Q_PROPERTY for QML
- Suboptimal connection types
- Performance issues (frequent updates)

## Automated Checks

```bash
# Find missing Q_OBJECT
grep -r "class.*:.*public QObject" --include="*.h" | \
  while read f; do grep -L "Q_OBJECT" "${f%:*}"; done

# Find old-style connections
grep -rn "SIGNAL\|SLOT" --include="*.cpp"

# Check MOC files are generated
ls moc_*.cpp
```

## Example Usage

```text
User: /qt-review

Agent:
# Qt Code Review Report

## Files Reviewed
- src/mainwindow.cpp (modified)
- src/worker.cpp (modified)
- qml/Main.qml (modified)

## Issues Found

[CRITICAL] GUI Access from Worker Thread
File: src/worker.cpp:45
Issue: Modifying QLabel from non-GUI thread
```cpp
void Worker::run() {
    // ...
    statusLabel->setText("Done");  // CRASH!
}
```
Fix: Use signal or QMetaObject::invokeMethod
```cpp
void Worker::run() {
    // ...
    emit workDone("Done");
}
// Connect in main thread:
connect(worker, &Worker::workDone, label, &QLabel::setText);
```

[HIGH] Old-style Connection
File: src/mainwindow.cpp:28
Issue: String-based signal/slot connection (no compile-time check)
```cpp
connect(button, SIGNAL(clicked()), this, SLOT(onClicked()));
```
Fix: Use new-style connection
```cpp
connect(button, &QPushButton::clicked, this, &MainWindow::onClicked);
```

[MEDIUM] Missing Q_PROPERTY
File: src/datamodel.h:15
Issue: Property exposed to QML without proper notification
```cpp
QString name() const { return m_name; }
```
Fix: Add Q_PROPERTY
```cpp
Q_PROPERTY(QString name READ name WRITE setName NOTIFY nameChanged)
```

## Summary
- CRITICAL: 1
- HIGH: 1
- MEDIUM: 1

Recommendation: Block merge until CRITICAL issue is fixed
```

## Approval Criteria

| Status | Condition |
|--------|-----------|
| Approve | No CRITICAL or HIGH issues |
| Warning | Only MEDIUM issues (merge with caution) |
| Block | CRITICAL or HIGH issues found |

## Qt Version Considerations

The agent checks for:
- Qt5 vs Qt6 API differences
- Deprecated Qt functions
- Modern Qt6 alternatives
- QML compatibility

## Integration with Other Commands

- Use `/cpp-review` for general C++ issues
- Use `/cpp-build` if MOC/build issues occur
- Use `/cpp-test` for Qt test framework

## Related

- Agent: `agents/qt-reviewer.md`
- Skills: `skills/qt-patterns/`
