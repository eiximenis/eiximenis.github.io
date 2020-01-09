---
title: 'ObservableCollection<T>, INotifyPropertyChanged y WinRT'
description: 'ObservableCollection<T>, INotifyPropertyChanged y WinRT'
author: eiximenis

date: 2011-10-16T12:35:00+00:00
geeks_url: /?p=1580
geeks_visits:
  - 1696
geeks_ms_views:
  - 1136
categories:
  - Uncategorized

---
**<span style="text-decoration: underline;">NOTA: Este post está basado en la versión Developers Preview de Windows 8, que salió en Septiembre del 2011. Versiones posteriores pueden dejar (y con suerte dejarán) este artículo obsoleto.</span>**

Un post cortito: Si desarrollas aplicaciones Metro para Windows 8 usando C# y XAML **no uses ObservableCollection<T>**. Simple y llanamente **no funciona**.

En su lugar debe usarse <a target="_blank" href="http://msdn.microsoft.com/en-us/library/windows/apps/br226052(v=vs.85).aspx" rel="noopener noreferrer">IObservableVector<T></a> interfaz de la cual podéis encontrar una implementación aquí: <http://code.msdn.microsoft.com/Data-Binding-7b1d67b5/sourcecode?fileId=44725&pathId=1428387049>. Esa implementación proporciona además un método ToObservableVector para convertir una <a target="_blank" href="http://msdn.microsoft.com/en-us/library/system.collections.specialized.inotifycollectionchanged.aspx" rel="noopener noreferrer">INotifyCollectionChanged</a> (es decir una ObservableCollection<T>) en un IObservableVector<T>.

Relacionado con el tema: **ojo con implementar INotifyPropertyChanged**. En concreto, ojo con _cual_ implementas pues resulta que ahora hay dos! Por un lado está el INotifyPropertyChanged _de toda la vida_ (<a target="_blank" href="http://msdn.microsoft.com/en-us/library/system.componentmodel.inotifypropertychanged.aspx" rel="noopener noreferrer">System.ComponentModel.INotifyPropertyChanged</a>) y por otro uno nuevo que es el que usa WinRT: <a target="_blank" href="http://msdn.microsoft.com/en-us/library/windows/apps/windows.ui.xaml.data.inotifypropertychanged" rel="noopener noreferrer">Windows.UI.Xaml.Data.INotifyPropertyChanged</a>. Ese último es el que tenéis que utilizar en vuestros ViewModels.

Desconozco el porque de estos cambios (no usar INotifyCollectionChanged ni el INotifyPropertyChanged de toda la vida), aunque supongo que tienen que ver en _no usar colecciones ni eventos propios de .NET_ y tenerlo todo controlado dentro del API de WinRT. También supongo que en siguientes versiones de Windows 8 eso se arreglará.

Así que si no queréis, como yo, perder un buen rato preguntándoos porque no se actualiza una ListBox... ya sabéis! 😉

Un saludo!

PD: Algunos enlaces que he encontrado buscando acerca de esto:

  1. ObservableCollection no funciona: <http://social.msdn.microsoft.com/Forums/en-AU/winappswithcsharp/thread/054913c2-6ad4-4b54-a349-c7ae846d4f8e>
  2. Selecciona el INotifyPropertyChanged correcto en WinRT: <http://blog.galasoft.ch/archive/2011/09/25/quick-tip-select-the-correct-inotifypropertychanged-in-windows-8.aspx> &ndash;> Según menciona aquí esto parece que ya está corregido y que la nueva versión de Win8 ya no tendrá ese error.