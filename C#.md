🔵 2. Markdown File: Full Guide for C# Transparent Docked Taskbar Animation

🎨 Creating a Transparent Animated Overlay Docked to the Windows Taskbar (C# Guide)

Full Step-by-Step Reference

📌 Overview

This document explains how to create a transparent, click-through, always-on-top WPF window that puts an animated GIF or image directly above the Windows taskbar, appearing as part of it.

This accomplishes the same visual effect as a Rainmeter skin, but fully custom in C#.

🧰 Requirements

Visual Studio

.NET 6/7/8

WPF Application template

Basic C# familiarity

📁 1. Create a WPF Project

Visual Studio →
File → New → Project → WPF App (.NET)

🪟 2. Configure Window for Transparency

Open MainWindow.xaml:

<Window x:Class="GhibliOverlay.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        WindowStyle="None"
        AllowsTransparency="True"
        Background="Transparent"
        Topmost="True"
        ShowInTaskbar="False"
        ResizeMode="NoResize"
        Loaded="Window_Loaded"
        Width="200" Height="200">
    <Image Name="AnimationPlayer"
           Width="120"
           Height="120"
           Source="totoro.gif"/>
</Window>


Purpose of each property:

WindowStyle=None → no title bar

AllowsTransparency=True → enables alpha blending

Background=Transparent → invisible background

Topmost=True → stays above taskbar

ShowInTaskbar=False → no icon in taskbar

Loaded event will install click-through behavior

🧱 3. Add Click-Through Behavior (Window does not block mouse)

MainWindow.xaml.cs:

using System;
using System.Runtime.InteropServices;
using System.Windows;
using System.Windows.Interop;

public partial class MainWindow : Window
{
    const int GWL_EXSTYLE = -20;
    const int WS_EX_TRANSPARENT = 0x20;
    const int WS_EX_LAYERED = 0x80000;

    [DllImport("user32.dll")]
    static extern int GetWindowLong(IntPtr hWnd, int nIndex);

    [DllImport("user32.dll")]
    static extern int SetWindowLong(IntPtr hWnd, int nIndex, int dwNewLong);

    public MainWindow()
    {
        InitializeComponent();
    }

    private void Window_Loaded(object sender, RoutedEventArgs e)
    {
        IntPtr hwnd = new WindowInteropHelper(this).Handle;

        int exStyle = GetWindowLong(hwnd, GWL_EXSTYLE);

        SetWindowLong(hwnd, GWL_EXSTYLE,
            exStyle | WS_EX_TRANSPARENT | WS_EX_LAYERED);

        DockToTaskbar();
    }
}

🪟 4. Detect Taskbar Position and Dock Animation Window

Add below inside the class:

[DllImport("shell32.dll")]
static extern IntPtr SHAppBarMessage(int dwMessage, ref APPBARDATA pData);

[StructLayout(LayoutKind.Sequential)]
public struct APPBARDATA
{
    public uint cbSize;
    public IntPtr hWnd;
    public uint uCallbackMessage;
    public uint uEdge;
    public RECT rc;
    public int lParam;
}

[StructLayout(LayoutKind.Sequential)]
public struct RECT
{
    public int left, top, right, bottom;
}


Now the docking logic:

private void DockToTaskbar()
{
    APPBARDATA abd = new APPBARDATA();
    abd.cbSize = (uint)Marshal.SizeOf(abd);

    // ABM_GETTASKBARPOS = 5
    SHAppBarMessage(0x00000005, ref abd);

    RECT rc = abd.rc;

    // Position just above taskbar:
    this.Left = rc.left + 10;     // slight padding from left side
    this.Top = rc.bottom - this.Height - 10; // just above taskbar
}

🎞 5. Add an Animation (GIF)

Add a GIF (e.g., totoro.gif) to your project:

Right-click project → Add → Existing Item

Select your GIF

Set Build Action = Resource

WPF automatically animates GIFs.

▶ 6. Run the Project

You now have:

transparent

click-through

always-on-top

taskbar-docked

Ghibli animation window.

To move it or interact with it, temporarily remove:

WS_EX_TRANSPARENT

🏁 End of Markdown File
