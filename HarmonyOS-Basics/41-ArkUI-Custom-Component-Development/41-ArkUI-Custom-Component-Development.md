# 41. ArkUI Custom Component Development

## Introduction

Custom component development is essential for creating reusable, maintainable, and scalable ArkUI applications. This article explores advanced techniques for building custom components, component composition patterns, and best practices for component architecture.

## Advanced Component Architecture

### Base Component Framework

```typescript
// Abstract base component with common functionality
@Component
export abstract struct BaseComponent {
  @Prop id?: string;
  @Prop className?: string;
  @Prop testId?: string;
  @State protected componentState: ComponentState = ComponentState.Idle;

  // Lifecycle hooks
  protected abstract onInit(): void;
  protected onMount(): void {}
  protected onUpdate(): void {}
  protected onUnmount(): void {}

  aboutToAppear() {
    this.componentState = ComponentState.Mounting;
    this.onInit();
    this.onMount();
    this.componentState = ComponentState.Mounted;
  }

  aboutToDisappear() {
    this.componentState = ComponentState.Unmounting;
    this.onUnmount();
    this.componentState = ComponentState.Unmounted;
  }

  // Abstract build method
  protected abstract buildContent(): void;

  build() {
    this.buildContent();
  }
}

enum ComponentState {
  Idle = 'idle',
  Mounting = 'mounting',
  Mounted = 'mounted',
  Updating = 'updating',
  Unmounting = 'unmounting',
  Unmounted = 'unmounted'
}

// Mixin pattern for component functionality
export interface ComponentMixin {
  mixinName: string;
  apply(component: any): void;
}

export class ValidationMixin implements ComponentMixin {
  mixinName = 'validation';

  apply(component: any): void {
    component.validate = (value: any, rules: ValidationRule[]) => {
      return this.validateValue(value, rules);
    };

    component.addValidationRule = (rule: ValidationRule) => {
      if (!component.validationRules) {
        component.validationRules = [];
      }
      component.validationRules.push(rule);
    };
  }

  private validateValue(value: any, rules: ValidationRule[]): ValidationResult {
    const errors: string[] = [];

    for (const rule of rules) {
      const result = rule.validator(value);
      if (!result.isValid) {
        errors.push(result.message);
      }
    }

    return {
      isValid: errors.length === 0,
      errors
    };
  }
}

interface ValidationRule {
  name: string;
  validator: (value: any) => { isValid: boolean; message: string };
}

interface ValidationResult {
  isValid: boolean;
  errors: string[];
}
```

### Advanced Form Components

```typescript
// Generic form field component with validation
@Component
export struct FormField {
  @Prop label: string = '';
  @Prop placeholder: string = '';
  @Prop required: boolean = false;
  @Prop disabled: boolean = false;
  @Prop fieldType: FieldType = FieldType.Text;
  @Link @Watch('onValueChange') value: string;
  @Prop validationRules: ValidationRule[] = [];
  @Prop customValidator?: (value: string) => ValidationResult;

  @State private isValid: boolean = true;
  @State private errors: string[] = [];
  @State private isTouched: boolean = false;
  @State private isFocused: boolean = false;

  private validationMixin = new ValidationMixin();

  aboutToAppear() {
    this.validationMixin.apply(this);
  }

  private onValueChange(): void {
    this.isTouched = true;
    this.validateField();
  }

  private validateField(): void {
    let validationResult: ValidationResult = { isValid: true, errors: [] };

    if (this.validationRules.length > 0) {
      validationResult = this.validate(this.value, this.validationRules);
    }

    if (this.customValidator && validationResult.isValid) {
      validationResult = this.customValidator(this.value);
    }

    this.isValid = validationResult.isValid;
    this.errors = validationResult.errors;
  }

  private getFieldColor(): Color {
    if (!this.isTouched) return Color.Gray;
    return this.isValid ? Color.Green : Color.Red;
  }

  build() {
    Column({ space: 8 }) {
      // Label
      if (this.label) {
        Row() {
          Text(this.label)
            .fontSize(14)
            .fontWeight(FontWeight.Medium)
            .fontColor(this.getFieldColor())

          if (this.required) {
            Text('*')
              .fontSize(14)
              .fontColor(Color.Red)
              .margin({ left: 4 })
          }
        }
        .width('100%')
        .justifyContent(FlexAlign.Start)
      }

      // Input field
      this.buildInputField()

      // Error messages
      if (!this.isValid && this.isTouched && this.errors.length > 0) {
        Column({ space: 4 }) {
          ForEach(this.errors, (error: string) => {
            Text(error)
              .fontSize(12)
              .fontColor(Color.Red)
              .width('100%')
          }, (error: string, index: number) => `error-${index}`)
        }
        .width('100%')
      }
    }
    .width('100%')
    .alignItems(HorizontalAlign.Start)
  }

  @Builder
  private buildInputField() {
    switch (this.fieldType) {
      case FieldType.Text:
      case FieldType.Email:
      case FieldType.Password:
        TextInput({
          text: this.value,
          placeholder: this.placeholder
        })
          .type(this.getInputType())
          .enabled(!this.disabled)
          .borderColor(this.getFieldColor())
          .borderWidth(this.isFocused ? 2 : 1)
          .onChange((text) => {
            this.value = text;
          })
          .onFocus(() => {
            this.isFocused = true;
          })
          .onBlur(() => {
            this.isFocused = false;
            this.isTouched = true;
            this.validateField();
          })
          .width('100%');
        break;

      case FieldType.TextArea:
        TextArea({
          text: this.value,
          placeholder: this.placeholder
        })
          .enabled(!this.disabled)
          .borderColor(this.getFieldColor())
          .borderWidth(this.isFocused ? 2 : 1)
          .onChange((text) => {
            this.value = text;
          })
          .onFocus(() => {
            this.isFocused = true;
          })
          .onBlur(() => {
            this.isFocused = false;
            this.isTouched = true;
            this.validateField();
          })
          .width('100%');
        break;

      case FieldType.Number:
        TextInput({
          text: this.value,
          placeholder: this.placeholder
        })
          .type(InputType.Number)
          .enabled(!this.disabled)
          .borderColor(this.getFieldColor())
          .borderWidth(this.isFocused ? 2 : 1)
          .onChange((text) => {
            this.value = text;
          })
          .onFocus(() => {
            this.isFocused = true;
          })
          .onBlur(() => {
            this.isFocused = false;
            this.isTouched = true;
            this.validateField();
          })
          .width('100%');
        break;
    }
  }

  private getInputType(): InputType {
    switch (this.fieldType) {
      case FieldType.Email:
        return InputType.Email;
      case FieldType.Password:
        return InputType.Password;
      case FieldType.Number:
        return InputType.Number;
      default:
        return InputType.Normal;
    }
  }
}

enum FieldType {
  Text = 'text',
  Email = 'email',
  Password = 'password',
  Number = 'number',
  TextArea = 'textarea'
}
```

### Complex Interactive Components

```typescript
// Advanced data table component
@Component
export struct DataTable<T> {
  @Prop columns: TableColumn<T>[] = [];
  @Prop data: T[] = [];
  @Prop selectable: boolean = false;
  @Prop sortable: boolean = true;
  @Prop filterable: boolean = true;
  @Prop pagination: boolean = true;
  @Prop pageSize: number = 10;
  @Prop onRowClick?: (row: T, index: number) => void;
  @Prop onSelectionChange?: (selectedRows: T[]) => void;

  @State private selectedRows: Set<number> = new Set();
  @State private currentPage: number = 1;
  @State private sortColumn: string = '';
  @State private sortDirection: 'asc' | 'desc' = 'asc';
  @State private filters: Record<string, string> = {};
  @State private filteredData: T[] = [];

  aboutToAppear() {
    this.filteredData = [...this.data];
    this.applyFiltersAndSort();
  }

  private applyFiltersAndSort(): void {
    let result = [...this.data];

    // Apply filters
    Object.keys(this.filters).forEach(columnKey => {
      const filterValue = this.filters[columnKey].toLowerCase();
      if (filterValue) {
        result = result.filter(row => {
          const cellValue = this.getCellValue(row, columnKey);
          return cellValue.toString().toLowerCase().includes(filterValue);
        });
      }
    });

    // Apply sorting
    if (this.sortColumn) {
      result.sort((a, b) => {
        const aValue = this.getCellValue(a, this.sortColumn);
        const bValue = this.getCellValue(b, this.sortColumn);

        let comparison = 0;
        if (aValue > bValue) comparison = 1;
        else if (aValue < bValue) comparison = -1;

        return this.sortDirection === 'desc' ? -comparison : comparison;
      });
    }

    this.filteredData = result;
  }

  private getCellValue(row: T, columnKey: string): any {
    return (row as any)[columnKey];
  }

  private toggleSort(columnKey: string): void {
    if (this.sortColumn === columnKey) {
      this.sortDirection = this.sortDirection === 'asc' ? 'desc' : 'asc';
    } else {
      this.sortColumn = columnKey;
      this.sortDirection = 'asc';
    }
    this.applyFiltersAndSort();
  }

  private toggleRowSelection(index: number): void {
    if (this.selectedRows.has(index)) {
      this.selectedRows.delete(index);
    } else {
      this.selectedRows.add(index);
    }

    if (this.onSelectionChange) {
      const selectedData = Array.from(this.selectedRows).map(i => this.filteredData[i]);
      this.onSelectionChange(selectedData);
    }
  }

  private getPaginatedData(): T[] {
    if (!this.pagination) return this.filteredData;

    const startIndex = (this.currentPage - 1) * this.pageSize;
    const endIndex = startIndex + this.pageSize;
    return this.filteredData.slice(startIndex, endIndex);
  }

  private getTotalPages(): number {
    return Math.ceil(this.filteredData.length / this.pageSize);
  }

  build() {
    Column({ space: 16 }) {
      // Filters (if enabled)
      if (this.filterable) {
        this.buildFilters();
      }

      // Table
      Column() {
        // Header
        Row() {
          if (this.selectable) {
            Checkbox({
              name: 'selectAll',
              group: 'tableHeader'
            })
              .select(this.selectedRows.size === this.filteredData.length)
              .onChange((isChecked) => {
                if (isChecked) {
                  this.filteredData.forEach((_, index) => {
                    this.selectedRows.add(index);
                  });
                } else {
                  this.selectedRows.clear();
                }
              })
              .width(40);
          }

          ForEach(this.columns, (column: TableColumn<T>) => {
            Row({ space: 4 }) {
              Text(column.title)
                .fontSize(14)
                .fontWeight(FontWeight.Bold)
                .layoutWeight(1);

              if (this.sortable && column.sortable !== false) {
                Text(this.getSortIndicator(column.key))
                  .fontSize(12)
                  .fontColor(Color.Gray);
              }
            }
            .width(column.width || '1fr')
            .padding({ horizontal: 8, vertical: 12 })
            .backgroundColor(Color.Gray)
            .onClick(() => {
              if (this.sortable && column.sortable !== false) {
                this.toggleSort(column.key);
              }
            });
          }, (column: TableColumn<T>) => column.key)
        }
        .width('100%');

        // Rows
        List() {
          ForEach(this.getPaginatedData(), (row: T, index: number) => {
            ListItem() {
              Row() {
                if (this.selectable) {
                  Checkbox({
                    name: `row-${index}`,
                    group: 'tableRows'
                  })
                    .select(this.selectedRows.has(index))
                    .onChange((isChecked) => {
                      this.toggleRowSelection(index);
                    })
                    .width(40);
                }

                ForEach(this.columns, (column: TableColumn<T>) => {
                  Text(this.formatCellValue(row, column))
                    .fontSize(14)
                    .width(column.width || '1fr')
                    .padding({ horizontal: 8, vertical: 12 })
                    .textOverflow({ overflow: TextOverflow.Ellipsis })
                    .maxLines(1);
                }, (column: TableColumn<T>) => column.key)
              }
              .width('100%')
              .backgroundColor(index % 2 === 0 ? Color.White : '#f9f9f9')
              .onClick(() => {
                if (this.onRowClick) {
                  this.onRowClick(row, index);
                }
              });
            }
          }, (row: T, index: number) => `row-${index}`)
        }
        .divider({
          strokeWidth: 1,
          color: Color.Gray,
          startMargin: 0,
          endMargin: 0
        });
      }
      .border({ width: 1, color: Color.Gray })
      .borderRadius(8);

      // Pagination (if enabled)
      if (this.pagination) {
        this.buildPagination();
      }
    }
    .width('100%');
  }

  @Builder
  private buildFilters() {
    Row({ space: 8 }) {
      ForEach(this.columns.filter(col => col.filterable !== false), (column: TableColumn<T>) => {
        TextInput({
          placeholder: `Filter ${column.title}`,
          text: this.filters[column.key] || ''
        })
          .onChange((text) => {
            this.filters[column.key] = text;
            this.applyFiltersAndSort();
          })
          .layoutWeight(1);
      }, (column: TableColumn<T>) => `filter-${column.key}`)
    }
    .width('100%');
  }

  @Builder
  private buildPagination() {
    Row({ space: 16 }) {
      Button('Previous')
        .enabled(this.currentPage > 1)
        .onClick(() => {
          this.currentPage--;
        });

      Text(`Page ${this.currentPage} of ${this.getTotalPages()}`)
        .fontSize(14);

      Button('Next')
        .enabled(this.currentPage < this.getTotalPages())
        .onClick(() => {
          this.currentPage++;
        });
    }
    .width('100%')
    .justifyContent(FlexAlign.Center);
  }

  private getSortIndicator(columnKey: string): string {
    if (this.sortColumn !== columnKey) return '↕️';
    return this.sortDirection === 'asc' ? '↑' : '↓';
  }

  private formatCellValue(row: T, column: TableColumn<T>): string {
    const value = this.getCellValue(row, column.key);

    if (column.formatter) {
      return column.formatter(value, row);
    }

    return value?.toString() || '';
  }
}

interface TableColumn<T> {
  key: string;
  title: string;
  width?: string;
  sortable?: boolean;
  filterable?: boolean;
  formatter?: (value: any, row: T) => string;
}
```

### Component Composition Patterns

```typescript
// Higher-order component pattern
export function withLoading<P extends Record<string, any>>(
  WrappedComponent: (props: P) => void
): (props: P & { loading?: boolean }) => void {
  return (props: P & { loading?: boolean }) => {
    if (props.loading) {
      Column({ space: 16 }) {
        LoadingProgress()
          .width(40)
          .height(40);
        Text('Loading...')
          .fontSize(14)
          .fontColor(Color.Gray);
      }
      .width('100%')
      .height('100%')
      .justifyContent(FlexAlign.Center);
    } else {
      WrappedComponent(props);
    }
  };
}

// Render props pattern
@Component
export struct RenderPropsContainer<T> {
  @Prop data: T;
  @Prop loading: boolean = false;
  @Prop error: string = '';
  @BuilderParam children: (data: T, loading: boolean, error: string) => void;

  build() {
    Column() {
      this.children(this.data, this.loading, this.error);
    }
    .width('100%')
    .height('100%');
  }
}

// Compound component pattern
@Component
export struct Modal {
  @Prop visible: boolean = false;
  @Prop onClose?: () => void;
  @BuilderParam content: () => void;

  build() {
    if (this.visible) {
      Stack() {
        // Backdrop
        Column()
          .width('100%')
          .height('100%')
          .backgroundColor(Color.Black)
          .opacity(0.5)
          .onClick(() => {
            if (this.onClose) {
              this.onClose();
            }
          });

        // Modal content
        Column() {
          this.content();
        }
        .backgroundColor(Color.White)
        .borderRadius(12)
        .padding(24)
        .margin(20);
      }
      .width('100%')
      .height('100%');
    }
  }
}

@Component
export struct ModalHeader {
  @Prop title: string = '';
  @Prop onClose?: () => void;

  build() {
    Row() {
      Text(this.title)
        .fontSize(20)
        .fontWeight(FontWeight.Bold)
        .layoutWeight(1);

      if (this.onClose) {
        Button('×')
          .type(ButtonType.Circle)
          .width(32)
          .height(32)
          .fontSize(20)
          .onClick(() => {
            this.onClose!();
          });
      }
    }
    .width('100%')
    .justifyContent(FlexAlign.SpaceBetween)
    .margin({ bottom: 16 });
  }
}

@Component
export struct ModalBody {
  @BuilderParam content: () => void;

  build() {
    Column() {
      this.content();
    }
    .width('100%')
    .layoutWeight(1);
  }
}

@Component
export struct ModalFooter {
  @BuilderParam actions: () => void;

  build() {
    Row({ space: 12 }) {
      this.actions();
    }
    .width('100%')
    .justifyContent(FlexAlign.End)
    .margin({ top: 16 });
  }
}
```

## Best Practices

1. **Component Composition**: Prefer composition over inheritance for component reusability
2. **Props Validation**: Implement proper validation for component props
3. **Event Handling**: Use consistent event naming and callback patterns
4. **Performance**: Optimize renders using proper state management
5. **Accessibility**: Include accessibility attributes and keyboard navigation
6. **Testing**: Write comprehensive tests for component behavior
7. **Documentation**: Provide clear documentation and usage examples

## Conclusion

Custom component development in ArkUI enables building sophisticated, reusable user interface elements. By leveraging advanced patterns like composition, mixins, and higher-order components, developers can create maintainable and scalable component libraries that enhance development productivity and application quality.

The key to successful custom component development lies in understanding component lifecycle, proper state management, and following established design patterns for maximum reusability and maintainability.
