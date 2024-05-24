Falls jemand auf die folgende Fehlermeldung stößt:

*"java.lang.NoSuchFieldError: Class com.sun.tools.javac.tree.JCTree$JCImport does not have member field 'com.sun.tools.javac.tree.JCTree qualid'"*

bedeutet dies, dass ein Versionskonflikt zwischen zwei Abhängigkeiten in eurer POM-Datei vorliegt. Ein häufiges Beispiel dafür ist die Einbindung von Lombok.

Lösung: Vermeidet die problematische Dependency und sucht nach einer einfacheren Alternative.
