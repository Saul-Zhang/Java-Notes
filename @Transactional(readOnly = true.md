```java

  /**
   * A boolean flag that can be set to true if the transaction is effectively read-only, allowing for corresponding optimizations at runtime.
   * Defaults to false.
   * This just serves as a hint for the actual transaction subsystem; it will not necessarily cause failure of write access attempts. A transaction manager which cannot interpret the read-only hint will not throw an exception when asked for a read-only transaction but rather silently ignore the hint.
   * See Also:
   * org.springframework.transaction.interceptor.TransactionAttribute.isReadOnly(), org.springframework.transaction.support.TransactionSynchronizationManager.isCurrentTransactionReadOnly()
   */
	boolean readOnly() default false;
```

